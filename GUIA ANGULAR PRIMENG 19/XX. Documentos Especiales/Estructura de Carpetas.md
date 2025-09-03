
## 📘 ¿De dónde viene esta estructura?

Esta estructura sigue principios de:

- **Feature-based architecture**: usada en Angular, React, Nx, NestJS, y promovida por el equipo Angular en guías y conferencias.
    
- **Clean Architecture adaptada a frontend**: separación de "core", "shared" y "features".
    
- **Angular Standalone Best Practices**: guía oficial + casos reales de empresas (puedo compartirte fuentes si quieres).
    
- Inspirada también en Nx Workspace Layout y conceptos de escalabilidad frontend aplicados en grandes equipos.


```
src/
├── app/                           # Punto de entrada principal de la aplicación
│   ├── app.component.ts           # Componente raíz standalone
│   ├── app.component.html         # Plantilla raíz (con <router-outlet>, <p-toast>, etc.)
│   ├── app.config.ts              # Configuración global (rutas, interceptores, providers)
│   ├── app.routes.ts              # Definición de rutas principales
│
│   ├── core/                      # Código base reutilizable, sin dependencia de dominio
│   │   ├── interceptors/          # Interceptores HTTP (errores, auth, logs, cache, etc.)
│   │   ├── guards/                # Guardias de rutas (auth, roles, etc.)
│   │   ├── services/              # Servicios globales (auth, notificaciones, temas, traducción)
│   │   ├── models/                # Tipos e interfaces TypeScript compartidos (DTOs, enums)
│   │   ├── tokens/                # Tokens de inyección o `HttpContextToken` globales
│   │   └── utils/                 # Funciones utilitarias (helpers, formateadores, validadores)
│
│   ├── shared/                    # Código reusable entre features
│   │   ├── components/            # Componentes reutilizables (ej. inputs, buscadores, widgets)
│   │   ├── directives/            # Directivas personalizadas (autofocus, permisos, etc.)
│   │   ├── pipes/                 # Pipes personalizados (capitalizar, currency, truncar, etc.)
│   │   └── shared.module.ts       # (opcional) Módulo para importar/exportar reutilizables y PrimeNG
│
│   ├── layout/                    # Estructura visual persistente (UI shell de la app)
│   │   ├── components/            # Sidebar, navbar, footer, breadcrumb, etc.
│   │   ├── styles/                # SCSS base (reset, tipografía, utilidades, mixins)
│   │   └── layout.config.ts       # Configuraciones del layout (temas, animaciones, dark/light)
│
│   ├── features/                  # Funcionalidades agrupadas por dominio (enfoque feature-first)
│   │   ├── auth/                  # Módulo de autenticación (login, register, etc.)
│   │   ├── dashboard/             # Módulo de panel principal
│   │   ├── usuarios/              # Módulo de gestión de usuarios
│   │   ├── productos/             # Módulo de productos (CRUD, filtros, stock, etc.)
│   │   └── ...                    # Otros módulos funcionales o dominios
│
│   ├── data/                      # Datos estáticos o de desarrollo local (mock JSON, constantes)
│   │   ├── config.json
│   │   └── datos-ejemplo.json
│
│   └── environments/              # Archivos de configuración por entorno
│       ├── environment.ts         # Entorno de desarrollo
│       └── environment.prod.ts    # Entorno de producción
│
├── assets/                        # Recursos estáticos (imágenes, íconos, fuentes, logos, etc.)
│   ├── img/
│   ├── icons/
│   └── styles/                    # CSS/SCSS global si no se usa CSS-in-JS
│
├── index.html                     # HTML base donde se monta Angular
├── main.ts                        # Bootstrap de la app
├── styles.scss                    # Estilos globales (si no se usa tailwind o external CSS)
└── tsconfig.json                  # Configuración TypeScript del proyecto

```

---

### ✅ Principales mejoras:

|Carpeta|Motivo|
|---|---|
|`core/`|Todo lo que se carga una vez y nunca cambia (interceptors, services globales, helpers, interfaces).|
|`shared/`|Componentes reutilizables, pipes, y utilidades comunes (ideal para toasts, loaders, etc).|
|`features/`|Sustituye a `pages/`, agrupa componentes, rutas y servicios por dominio funcional.|
|`layout/`|Para header/sidebar/footer, estilos base y temas (lo que ya tienes está muy bien aquí).|
|`data/`|Data estática o mocks para pruebas locales.|

### 🧠 Tips adicionales

- Usa `@` como alias de path (`tsconfig.json`), p. ej. `@/app/core/services/auth.service.ts`.
    
- Documenta brevemente cada carpeta en un `README.md` si el proyecto escala con más personas.
    
- Divide los servicios API por dominio si crecen, y colócalos en `features/<modulo>/services/`.
    
