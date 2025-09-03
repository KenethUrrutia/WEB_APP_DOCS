## 1. 🧱 `p-toast` global

Agrega **una sola vez** el componente `<p-toast>` en `app.component.html`:

```html
<!-- app.component.html -->
<p-toast key="global" position="top-right" [life]="5000"></p-toast>
<router-outlet />
```

Y en `app.component.ts`, importa el módulo visual:

```ts
import { Component } from '@angular/core';
import { ToastModule } from 'primeng/toast';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  templateUrl: './app.component.html',
  imports: [RouterOutlet, ToastModule],
})
export class AppComponent {}
```

## 2. 🧩 Configuración global en `app.config.ts`

Aquí defines los `providers` y registras interceptores:

```ts
// Otros imports
import { ToastModule } from 'primeng/toast';
import { MessageService } from 'primeng/api';

import { ErrorInterceptor } from './core/interceptors/error.interceptor';

export const appConfig = {
  providers: [
    
    
    provideHttpClient(withInterceptors([ErrorInterceptor, OtroInterceptor])),
    importProvidersFrom(ToastModule),
    MessageService,
  ]
};
```

---

## 3. 📣 Servicio de notificaciones (`notificaciones.service.ts`)

```ts
import { Injectable } from '@angular/core';
import { MessageService } from 'primeng/api';

@Injectable({ providedIn: 'root' })
export class NotificacionesService {
  constructor(private messageService: MessageService) {}

  success(summary: string, detail?: string) {
    this.messageService.add({ key: 'global', severity: 'success', summary, detail });
  }

  error(summary: string, detail?: string) {
    this.messageService.add({ key: 'global', severity: 'error', summary, detail });
  }

  info(summary: string, detail?: string) {
    this.messageService.add({ key: 'global', severity: 'info', summary, detail });
  }

  warn(summary: string, detail?: string) {
    this.messageService.add({ key: 'global', severity: 'warn', summary, detail });
  }
}
```

---

## 4. 🚦 Interceptor de errores (`error.interceptor.ts`)

Define  los tokens para contexto en `http-context.tokens.ts`:

```ts
// http-context.tokens.ts
import { HttpContextToken } from '@angular/common/http';

export const SKIP_GLOBAL_ERROR = new HttpContextToken<boolean>(() => false);
export const CUSTOM_ERROR_MESSAGE = new HttpContextToken<string | null>(() => null);
```

Interceptor funcional y standalone:

```ts
// error.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';

import { CUSTOM_ERROR_MESSAGE, SKIP_GLOBAL_ERROR } from './http-context.tokens';
import { NotificacionesService } from '../services/notificaciones.service';

export const ErrorInterceptor: HttpInterceptorFn = (req, next) => {
  const notificaciones = inject(NotificacionesService);
  const skip = req.context.get(SKIP_GLOBAL_ERROR);
  const customMessage = req.context.get(CUSTOM_ERROR_MESSAGE);

  if (skip) return next(req);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      const mensajes: Record<number, string> = {
        401: 'No autorizado.',
        403: 'Acceso denegado.',
        404: 'Recurso no encontrado.',
        500: 'Error del servidor.',
      };

      const resumen = customMessage || mensajes[error.status] || 'Error inesperado.';
      const detalle = error.error?.message || error.message || 'Ocurrió un error.';

      notificaciones.error(resumen, detalle);

      return throwError(() => error);
    })
  );
};
```

---

## 5. 📌 Cómo personalizar errores desde componentes (vía servicios)

### ✅ a) En el componente:

```ts
this.pagosService.getHistorialSalario({
  context: new HttpContext()
    .set(CUSTOM_ERROR_MESSAGE, 'No se pudo cargar el historial de salarios.')
})
.subscribe({
  next: (data) => { ... },
  error: (e) => {
    // Opcional: lógica personalizada del componente
  }
});
```

### ✅ b) En el servicio (ej. `pagos.service.ts`):

```ts
getHistorialSalario(options?: { context?: HttpContext }) {
  return this.http.get('/api/historial-salario', {
    context: options?.context || new HttpContext()
  });
}
```

---

## 6. 🔕 Omitir el interceptor en una petición

Desde el componente:

```ts
this.pagosService.getHistorialSalario({
  context: new HttpContext().set(SKIP_GLOBAL_ERROR, true)
}).subscribe({
  next: (data) => { ... },
  error: () => {
    // El interceptor NO muestra toast
    this.notificaciones.error('Error personalizado local');
  }
});
```

---

## 7. ✅ Mostrar mensajes manuales sin errores HTTP

```ts
this.notificaciones.success('Guardado', 'Los datos se guardaron correctamente');
```

---

## 🧠 Buenas prácticas

- Proveer `MessageService` solo una vez (en `app.config.ts`).
    
- Mantener los servicios de UI desacoplados (como `NotificacionesService`).
    
- No atrapes todos los errores en los componentes, deja que el interceptor actúe por defecto.
    
- Solo usa `SKIP_GLOBAL_ERROR` o `CUSTOM_ERROR_MESSAGE` cuando necesites un comportamiento especial.
    
- Usa interceptores **funcionales** (`HttpInterceptorFn`) para un diseño más limpio con standalone.
    
