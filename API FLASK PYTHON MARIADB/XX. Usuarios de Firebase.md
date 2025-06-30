# Guía Técnica: Autenticación con Firebase (Email/Contraseña) y JWT personalizados (Angular 19 + PrimeNG / Flask-RESTX)

Esta guía describe paso a paso cómo usar **Firebase Authentication** (solo email/contraseña) como proveedor de identidad en una aplicación con frontend en Angular 19 Standalone (+ PrimeNG) y backend en Python/Flask (Flask-RESTX). El backend manejará la autorización, emisión/validación de JWT y gestión de roles usando SQLAlchemy/Marshmallow/MySQL. Se incluyen ejemplos de código completos y referencias oficiales.

## 1. Configuración de Firebase Authentication (Email/Contraseña)

1. **Crear proyecto en Firebase:** Accede a [Firebase Console](https://console.firebase.google.com/) y crea un nuevo proyecto. Anota la configuración de Firebase (API Key, Auth Domain, etc.).
    
2. **Habilitar proveedor Email/Contraseña:** En la Firebase Console, ve a **Authentication → Sign-in method**, selecciona **Email/Password** y actívalo. Esto permite crear usuarios con correo y contraseña.
    
3. **Obtener credenciales del servicio (Admin):** Para el backend, en Firebase Console ve a **Project Settings → Service Accounts** y genera una nueva clave privada (JSON). Descarga este archivo (`serviceAccountKey.json`) y guárdalo en el servidor seguro. Se usará para inicializar `firebase-admin`.
    
4. **Dependencias:**
    
    - Frontend (Angular): `npm install firebase @angular/fire primeicons primeng` (PrimeNG para UI).
        
    - Backend (Python): `pip install firebase-admin Flask-RESTX flask-sqlalchemy marshmallow PyJWT`.
        

## 2. Frontend (Angular 19 Standalone + PrimeNG)

### 2.1. Inicialización de Firebase en Angular

Con componentes standalone (Angular 15+), inicializa Firebase en `main.ts` usando AngularFire **sin NgModules**. Por ejemplo:

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { provideFirebaseApp } from '@angular/fire/app';
import { provideAuth } from '@angular/fire/auth';

const firebaseConfig = { /* datos de tu proyecto Firebase */ };

bootstrapApplication(AppComponent, {
  providers: [
    // Inicializar Firebase App y Auth
    provideFirebaseApp(() => initializeApp(firebaseConfig)),
    provideAuth(() => getAuth())
    // ...otros providers (rutas, interceptores, etc.)
  ]
});
```

Esta configuración importa e inicia Firebase y el servicio de Auth. (En lugar de AngularFire podemos usar directamente `firebase/auth`, pero AngularFire facilita la inyección).

### 2.2. Componente de Login con PrimeNG

Crea un componente Standalone de login usando PrimeNG para la interfaz. Ejemplo básico:

```html
<!-- login.component.html -->
<p-panel header="Iniciar Sesión">
  <div class="p-fluid">
    <div class="p-field">
      <label for="email">Correo:</label>
      <input id="email" type="text" pInputText [(ngModel)]="email" />
    </div>
    <div class="p-field">
      <label for="password">Contraseña:</label>
      <input id="password" type="password" pPassword [(ngModel)]="password" feedback="false"/>
    </div>
    <button pButton type="button" label="Entrar" (click)="login()" class="p-mt-2"></button>
  </div>
</p-panel>
```

```typescript
// login.component.ts
import { Component } from '@angular/core';
import { signInWithEmailAndPassword, getAuth } from 'firebase/auth';
import { HttpClient } from '@angular/common/http';

@Component({
  standalone: true,
  selector: 'app-login',
  templateUrl: './login.component.html',
  imports: [ /* PrimeNG modules, FormsModule, etc. */ ]
})
export class LoginComponent {
  email = '';
  password = '';
  constructor(private http: HttpClient) {}

  async login() {
    try {
      const auth = getAuth();
      const userCred = await signInWithEmailAndPassword(auth, this.email, this.password);
      const idToken = await userCred.user.getIdToken(); // JWT de Firebase
      // Enviar token al backend para obtener JWT personalizado
      const res = await this.http.post('/api/auth/login', { id_token: idToken }).toPromise();
      localStorage.setItem('access_token', (res as any).access_token);
      // redirigir a ruta protegida...
    } catch (err) {
      console.error('Error de login', err);
    }
  }
}
```

Este componente usa `signInWithEmailAndPassword` de Firebase Auth para autenticar. Al realizar el login, llama a `getIdToken()` para obtener el JWT (ID Token) de Firebase del usuario. Luego envía ese token al backend para obtener un token personalizado (ver sección 6).

### 2.3. Envío del Token en cada petición (HttpInterceptor)

Para que el backend reciba el token en cada petición, implementa un **HttpInterceptor** que inserte la cabecera `Authorization: Bearer <token>` en cada request. Ejemplo:

```typescript
// auth.interceptor.ts
import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { from, Observable } from 'rxjs';
import { mergeMap } from 'rxjs/operators';
import { getAuth } from 'firebase/auth';

@Injectable({ providedIn: 'root' })
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Obtener token actual (ID token de Firebase)
    const auth = getAuth();
    // Esperar el token (firebase refresca automáticamente si expiró)
    return from(auth.currentUser?.getIdToken() ?? Promise.resolve(null)).pipe(
      mergeMap(token => {
        if (token) {
          // Clonar request e insertar header Authorization
          req = req.clone({
            setHeaders: { Authorization: `Bearer ${token}` }
          });
        }
        return next.handle(req);
      })
    );
  }
}
```

Este código usa `firebase/auth` directamente y obtiene el token del usuario autenticado. Como vemos en ejemplos similares, se captura el token y se añade a los headers de cada petición. Registra este interceptor en `bootstrapApplication` o en `AppComponent` providers para que se aplique globalmente.

### 2.4. Guard de Rutas (Opcional)

Para proteger rutas en el cliente, se puede usar un **AuthGuard** que verifique la existencia de un token válido antes de permitir acceder. Por ejemplo:

```typescript
// auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';

@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(private router: Router) {}
  canActivate(): boolean {
    const token = localStorage.getItem('access_token');
    if (!token) {
      this.router.navigate(['/login']);
      return false;
    }
    // Opcional: validar estructura o decodificar JWT localmente (no es infalible)
    return true;
  }
}
```

Este guard simplemente comprueba que exista un token almacenado. Para mayor seguridad se pueden decodificar los claims y verificar `exp`, pero la validación definitiva la hará el backend.


## 6. Ejemplos de Código

### 6.1. Angular: Servicio de Autenticación

```typescript
// auth.service.ts (Angular Standalone, usando Firebase Auth)
import { Injectable } from '@angular/core';
import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword } from 'firebase/auth';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private auth = getAuth();

  async signup(email: string, password: string) {
    return await createUserWithEmailAndPassword(this.auth, email, password);
  }

  async login(email: string, password: string) {
    return await signInWithEmailAndPassword(this.auth, email, password);
  }

  logout() {
    this.auth.signOut();
    localStorage.removeItem('access_token');
  }
}
```

### 6.2. Angular: Guard y Componente

```typescript
// profile.guard.ts
import { CanActivate, Router } from '@angular/router';

@Injectable({ providedIn: 'root' })
export class ProfileGuard implements CanActivate {
  constructor(private router: Router) {}
  canActivate(): boolean {
    if (localStorage.getItem('access_token')) return true;
    this.router.navigate(['/login']);
    return false;
  }
}
```

```html
<!-- profile.component.html (Protected) -->
<h2>Perfil de usuario</h2>
<p>Bienvenido al área segura.</p>
```

```typescript
// profile.component.ts
@Component({ standalone: true, templateUrl: './profile.component.html' })
export class ProfileComponent {}
```

Configura las rutas para usar el guard:

```typescript
// app-routing.ts
import { ProfileGuard } from './profile.guard';
const routes: Routes = [
  { path: 'login', component: LoginComponent },
  { path: 'profile', component: ProfileComponent, canActivate: [ProfileGuard] }
];
```

