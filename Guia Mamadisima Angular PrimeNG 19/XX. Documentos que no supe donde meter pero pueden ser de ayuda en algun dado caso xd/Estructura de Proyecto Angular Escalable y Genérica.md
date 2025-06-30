
Esta estructura sigue principios consolidados en el desarrollo frontend moderno, combinando patrones arquitectónicos adaptados para Angular:

- **Arquitectura basada en funcionalidades (feature-based architecture)**: fomenta encapsulamiento por dominio.
- **Clean Architecture adaptada al frontend**: separación entre capas sin acoplamientos circulares.
- **Mejores prácticas Angular Standalone**: uso de componentes, rutas y providers independientes sin módulos acoplados.
- **Layout de Nx Workspace**: organización clara para escalar en equipos grandes y múltiples dominios.

---

## Estructura Propuesta

```

src/  
├── app/ # Entrada principal de la aplicación  
│ ├── app.component.ts
│ ├── app.component.html 
│ ├── app.config.ts
│ ├── app.routes.ts 
│  
│ ├── core/ # Funcionalidad base sin dependencia de dominio  
│ │ ├── interceptors/
│ │ ├── guards/
│ │ ├── services/ # Servicios globales  
│ │ ├── interfaces/ # Interfaces y tipos  
│ │ ├── tokens/ # Tokens de inyección  
│ │ └── utils/
│  
│ ├── shared/ # Elementos reutilizables  
│ │ ├── components/ # Componentes genéricos  
│ │ ├── directives/ # Directivas personalizadas  
│ │ ├── pipes/ # Pipes comunes  
│ │ └── shared.module.ts # Import/export de componentes compartidos (opcional)  
│  
│ ├── layout/ # Estructura visual común  
│ │ ├── components/ # Elementos persistentes (nav, sidebar, etc.)  
│ │ ├── styles/ # SCSS base  
│ │ └── layout.config.ts # Configuraciones visuales  
│  
│ ├── features/ # Funcionalidades agrupadas por dominio  
│ │ ├── dominio-a/  
│ │ ├── dominio-b/  
│ │ └── ...  
│  
│ ├── data/ # Datos de prueba o configuración estática  
│ │ ├── config.json  
│ │ └── mock-data.json  
│  
│ └── environments/ # Configuración por entorno  
│ ├── environment.ts  
│ └── environment.prod.ts  
│  
├── assets/ # Recursos estáticos  
│ ├── img/  
│ ├── icons/  
│ └── styles/  
│  
├── index.html  
├── main.ts  
├── styles.scss  
└── tsconfig.json

```

---

## Tabla de Propósito de Carpetas

| Carpeta        | Descripción                                                                 |
|----------------|------------------------------------------------------------------------------|
| `core/`        | Código cargado una sola vez en el arranque. No depende de funcionalidades.  |
| `shared/`      | Elementos reutilizables en múltiples contextos (UI, pipes, directivas).     |
| `features/`    | Componentes, servicios y rutas agrupados por dominio funcional.             |
| `layout/`      | Estructura visual persistente de la aplicación.                             |
| `data/`        | Datos estáticos o mocks para desarrollo.                                    |
| `environments/`| Variables específicas por entorno.                                           |
| `assets/`      | Recursos externos no procesados por Angular (imágenes, fuentes, etc.).      |

---

## Buenas Prácticas y Patrones Recomendados

- **Alias de paths** en `tsconfig.json`:  
  ```json
  "paths": {
    "@/*": ["src/app/*"],
    "@core/*": ["src/app/core/*"],
    "@shared/*": ["src/app/shared/*"]
  }
```

Los alias de paths en `tsconfig.json` permiten definir rutas simbólicas para importar módulos sin usar rutas relativas largas o frágiles. Ejemplo:

```ts
// Sin alias
import { AuthService } from '../../../../core/services/auth.service';

// Con alias
import { AuthService } from '@core/services/auth.service';
```

Configurar en `tsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["app/*"],
      "@core/*": ["app/core/*"],
      "@shared/*": ["app/shared/*"],
      "@features/*": ["app/features/*"]
    }
  }
}
```

- **Separación estricta de responsabilidades**:
    
    - `core` contiene elementos fundamentales que no deben depender de ninguna lógica de dominio. Se carga una vez y permanece estable (interceptores, guardias, servicios transversales).
        
    - `shared` ofrece componentes y utilidades reutilizables, sin lógica de negocio ni dependencia de dominios funcionales.
        
- **Rutas standalone por dominio**:  
    Las rutas usan `loadComponent` en lugar de módulos, eliminando la necesidad de `NgModules`. Esto reduce acoplamiento, mejora el lazy loading y acelera el arranque.
    
    ```ts
    {
      path: 'usuarios',
      loadComponent: () => import('./features/usuarios/lista.component').then(m => m.ListaComponent)
    }
    ```
    
- **Inyección de dependencias desacoplada (tokens)**:  
    Define interfaces o tokens personalizados para servicios compartidos. Permite reemplazar implementaciones sin romper dependencias.
    
    ```ts
    export const LOGGER = new InjectionToken<Logger>('Logger');
    ```
    
- **Servicios por dominio** dentro de `features/<dominio>/services/`:  
    Encapsula los servicios relacionados con una funcionalidad dentro de su carpeta. Facilita mantenimiento, pruebas y aislamiento de lógica.
    
- **Componentes autocontenidos**:  
    Cada componente tiene su propia plantilla, lógica y estilos. Evita acoplamientos implícitos. Mejora reutilización y legibilidad.
    
- **Signals, inputs standalone, zoneless strategy**:
    
    - **Signals**: para estado reactivo sin `RxJS`.
        
    - **Standalone inputs/components**: evitan necesidad de módulos.
        
    - **Zoneless strategy**: desactiva `zone.js` para mejorar performance, requiere uso explícito de detección de cambios.
        
- **Documentación local mínima**:  
    Un `README.md` por carpeta explicando qué contiene y cómo interactúa con otras capas. Útil en equipos grandes o rotación frecuente.
    
- **Evita singletons innecesarios**:  
    Inyecta servicios solo en el contexto donde se usan. Evita `providedIn: 'root'` por defecto. Reduce carga global y facilita pruebas aisladas.
    
- **Evita estructuras rígidas por convención**:  
    No fuerces carpetas como `components`, `pages`, `services` si el dominio no lo requiere. Agrupa por comportamiento, no por tipo de archivo.


