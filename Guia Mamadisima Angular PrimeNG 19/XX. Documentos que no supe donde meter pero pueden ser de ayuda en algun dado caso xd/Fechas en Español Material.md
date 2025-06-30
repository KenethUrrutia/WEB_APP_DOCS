Este documento describe cómo configurar correctamente el soporte de fechas en español (`es` o `es-GT`) en una aplicación Angular 19 que usa componentes standalone

---

## 1. Registrar el locale español globalmente

En el archivo `app.config.ts`, registrar el locale español para que Angular pueda usar formatos de fecha en español:

```ts
// app.config.ts

import localeEs from '@angular/common/locales/es';
import { registerLocaleData } from '@angular/common';

registerLocaleData(localeEs, 'es');
```

---

## 2. Formato personalizado para Angular Material Datepicker

Crear un archivo de configuración con el formato de fechas personalizado. Por ejemplo: `src/app/core/utils/date-format.utils.ts`

```ts
// date-format.utils.ts

export const MY_FORMATS = {
  parse: {
    dateInput: 'LL'
  },
  display: {
    dateInput: 'LL',
    monthYearLabel: 'MMM YYYY',
    dateA11yLabel: 'LL',
    monthYearA11yLabel: 'MMMM YYYY'
  }
};
```

---

## 3. Configurar el componente con MAT_DATE_LOCALE y el adaptador

En cada componente standalone donde uses `MatDatepicker`, configura el locale y los formatos:

```ts
import { Component } from '@angular/core';
import { DatePipe } from '@angular/common';
import { MAT_DATE_LOCALE } from '@angular/material/core';
import { provideMomentDateAdapter } from '@angular/material-moment-adapter';
import { MY_FORMATS } from '../../shared/utils/date-format.utils';

@Component({
  selector: 'app-nuevo-usuario',
  standalone: true,
  imports: [],
  templateUrl: './nuevo-usuario.component.html',
  styleUrl: './nuevo-usuario.component.css',
  providers: [
    { provide: MAT_DATE_LOCALE, useValue: 'es-GT' },
    provideMomentDateAdapter(MY_FORMATS),
    DatePipe
  ]
})
export default class NuevoUsuarioComponent {
  constructor(private datePipe: DatePipe) {}
}
```

---

## 4. Crear pipes personalizados para formatear fechas y horas

Pipes para mostrar fechas y horas en español, reutilizables en plantillas HTML.

```ts
// date-format.pipe.ts

import { DatePipe } from '@angular/common';
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'dateFormat',
  standalone: true,
})
export class DateFormatPipe implements PipeTransform {
  transform(value: string | Date): string | null {
    const datePipe = new DatePipe('es');
    const date = new Date(value);
    return datePipe.transform(date, "d 'de' MMMM 'de' y");
  }
}
```

```ts
// time-format.pipe.ts

import { DatePipe } from '@angular/common';
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'timeFormat',
  standalone: true,
})
export class TimeFormatPipe implements PipeTransform {
  transform(value: string): string | null {
    const datePipe = new DatePipe('es');
    const date = new Date(value);
    return datePipe.transform(date, 'HH:mm a');
  }
}
```

---

## Resultado

- Angular usa fechas localizadas en español en componentes y pipes.
    
- `MatDatepicker` muestra y parsea en el formato `LL`.
    
- Las fechas y horas se presentan como "1 de junio de 2025" y "14:30 p. m." según corresponda.
    

---


