# Guía completa de SheetJS (xlsx) en Angular

SheetJS (paquete [`xlsx` en NPM](https://www.npmjs.com/package/xlsx)) es una librería JavaScript para crear y leer archivos Excel en navegadores y Node. Permite generar libros de trabajo (_workbooks_) con múltiples hojas, importar datos desde tablas HTML o JSON, y configurar propiedades avanzadas (celdas combinadas, ancho de columnas, fórmulas, etc.). A continuación se muestra un recorrido detallado paso a paso con ejemplos en TypeScript (Angular Standalone 17+). Cada sección incluye código de ejemplo y referencias a la documentación oficial para ilustrar cómo usar las distintas funcionalidades.

## Instalación y configuración

Instala la librería desde NPM en tu proyecto Angular:

```bash
npm install xlsx --save
```

Luego, en tu componente o servicio Angular, importa las utilidades que vas a usar. Por ejemplo, para exportar datos es común hacer:

```ts
import { Component } from '@angular/core';
import { read, utils, writeFileXLSX } from 'xlsx';
```

o bien importar todo con `import * as XLSX from 'xlsx';`. Ambas formas funcionan en Angular 17+. En entornos con módulos ESM puede ser necesario usar las funciones específicas (`utils.book_new`, `utils.json_to_sheet`, etc.) según la documentación.

## Crear un nuevo archivo Excel (Workbook)

Para crear un archivo desde cero, primero crea un nuevo libro de trabajo (workbook) vacío con `XLSX.utils.book_new()`, y luego añade hojas de cálculo (_worksheets_) con `XLSX.utils.book_append_sheet()`. Por ejemplo:

```ts
const wb = XLSX.utils.book_new(); 
const ws = XLSX.utils.aoa_to_sheet([   // crea una hoja a partir de array de arrays
  ["Nombre", "Edad"],
  ["Alice", 30],
  ["Bob", 25]
]);
XLSX.utils.book_append_sheet(wb, ws, 'Personas');  // agrega la hoja al libro
XLSX.writeFileXLSX(wb, 'mi_archivo.xlsx');         // descarga/genera el archivo
```

- `book_new()` crea un libro nuevo.
    
- `aoa_to_sheet(data)` convierte un array de arrays (cada sub-array es una fila) en un objeto hoja de cálculo. Alternativamente, `json_to_sheet(objetos)` convierte un arreglo de objetos en hoja usando las claves como encabezados.
    
- `book_append_sheet(workbook, worksheet, nombre)` añade la hoja al libro con un nombre dado.
    
- Finalmente `writeFileXLSX(workbook, filename)` descarga el XLSX al cliente. En entornos de NodeJS, `writeFile` escribiría el archivo localmente.
    

**Tabla: Funciones principales de XLSX.utils** (SheetJS, utilidades)

| Función / Propiedad                     | Descripción                                                                                                                        | Ejemplo de uso                                                                        |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `utils.book_new()`                      | Crea un nuevo libro de trabajo (workbook) vacío.                                                                                   | `const wb = XLSX.utils.book_new();`                                                   |
| `utils.book_append_sheet(wb, ws, name)` | Añade la hoja `ws` al libro `wb` con nombre `name`.                                                                                | `XLSX.utils.book_append_sheet(wb, ws, 'Hoja1');`                                      |
| `utils.aoa_to_sheet(array)`             | Convierte un _array de arrays_ en una hoja (cada sub-array es una fila).                                                           | `const ws = XLSX.utils.aoa_to_sheet([[1,2],[3,4]]);`                                  |
| `utils.json_to_sheet(arrayObjetos)`     | Convierte un arreglo de objetos JSON a hoja. Usa las claves de objeto como encabezados.                                            | `const ws = XLSX.utils.json_to_sheet([{Nom:'Ana', Edad:28}, {Nom:'Luis', Edad:35}]);` |
| `utils.sheet_add_aoa(ws, aoa, opts)`    | Inserta datos (array de arrays) en hoja existente, útil para añadir o reemplazar filas. Por ejemplo para sobrescribir encabezados. | `XLSX.utils.sheet_add_aoa(ws, [["Nombre","Edad"]], { origin: "A1" });`                |
| `utils.table_to_sheet(tableElem)`       | Convierte una tabla DOM (`<table>`) en una hoja.                                                                                   | `const ws = XLSX.utils.table_to_sheet(document.getElementById('miTabla'));`           |
| `utils.table_to_book(tableElem)`        | Convierte una tabla DOM en un libro de trabajo completo. Contiene por defecto una hoja con los datos de la tabla.                  | `const wb = XLSX.utils.table_to_book(document.getElementById('miTabla'));`            |
| `writeFileXLSX(workbook, filename)`     | Exporta/descarga el `workbook` como archivo .xlsx.                                                                                 | `XLSX.writeFileXLSX(wb, 'datos.xlsx');`                                               |

## Exportar datos desde una tabla HTML

Si ya tienes una tabla (`<table>`) en el DOM, puedes convertirla directamente en un Excel. Por ejemplo, en tu plantilla Angular:

```html
<table id="tablaUsuarios">
  <tr><th>Nombre</th><th>Edad</th></tr>
  <tr><td>Ana</td><td>28</td></tr>
  <tr><td>Luis</td><td>35</td></tr>
</table>
<button (click)="exportarDesdeTabla()">Exportar XLSX</button>
```

Y en tu componente TypeScript:

```ts
import { Component } from '@angular/core';
import * as XLSX from 'xlsx';

@Component({ /* ... */ })
export class MiComponente {
  exportarDesdeTabla() {
    const tabla = document.getElementById('tablaUsuarios') as HTMLTableElement;
    if (!tabla) { return; }
    // Opción A: crear un libro con una hoja de la tabla
    const wb = XLSX.utils.table_to_book(tabla); 
    XLSX.writeFile(wb, 'usuarios_tabla.xlsx');
    
    // Opción B: convertir sólo la hoja y manejar el libro manualmente:
    // const ws = XLSX.utils.table_to_sheet(tabla);
    // const wb = XLSX.utils.book_new();
    // XLSX.utils.book_append_sheet(wb, ws, 'Hoja1');
    // XLSX.writeFile(wb, 'usuarios_tabla.xlsx');
  }
}
```

- `table_to_sheet(tabla)` extrae los datos de la tabla DOM a una hoja.
    
- `table_to_book(tabla)` hace lo mismo pero crea y retorna directamente un _workbook_ con esa hoja incorporada.
    
- Finalmente usamos `writeFile` (o `writeFileXLSX`) para descargar. Los datos numéricos se detectan automáticamente como números.
    

## Exportar desde un arreglo de objetos (JSON)

Un caso común es tener datos en un arreglo de objetos JavaScript y querer exportarlos. La librería soporta esto directamente:

```ts
import { Component } from '@angular/core';
import { utils, writeFileXLSX } from 'xlsx';

@Component({ /* ... */ })
export class MiComponente {
  datos = [
    { Nombre: "Ana", Edad: 28 },
    { Nombre: "Luis", Edad: 35 }
  ];
  exportarJSON() {
    // Convierte el arreglo de objetos a una hoja de cálculo
    const ws = utils.json_to_sheet(this.datos);
    // Crea el libro y agrega la hoja
    const wb = utils.book_new();
    utils.book_append_sheet(wb, ws, 'Usuarios');
    // Opcional: sobrescribir encabezados si se desea (por defecto usa las claves)
    // utils.sheet_add_aoa(ws, [["Nombre", "Edad"]], { origin: "A1" });
    // Exportar archivo
    writeFileXLSX(wb, 'usuarios_json.xlsx');
  }
}
```

Por defecto `json_to_sheet` crea la primera fila con los encabezados basados en las claves de los objetos (p.ej. `"Nombre"`, `"Edad"`). Si quieres usar títulos diferentes o en otro idioma, puedes sobrescribir esa fila usando `sheet_add_aoa`, como se muestra arriba. Por ejemplo, para cambiar “Nombre” a “Nombre completo”:

```ts
// Después de json_to_sheet y antes de writeFile:
XLSX.utils.sheet_add_aoa(ws, [["Nombre completo", "Años"]], { origin: "A1" });
```

Esto reemplaza la primera fila (origen `"A1"`) con los valores provistos.

## Varias hojas (sheets) en un mismo archivo

Puedes agregar múltiples hojas al libro llamando `book_append_sheet` cada vez con un nombre distinto. Ejemplo:

```ts
const wb = XLSX.utils.book_new();
const ws1 = XLSX.utils.aoa_to_sheet([["A1", 1]]);
const ws2 = XLSX.utils.aoa_to_sheet([["B1", 2]]);
XLSX.utils.book_append_sheet(wb, ws1, 'Hoja1');
XLSX.utils.book_append_sheet(wb, ws2, 'Hoja2');
// Escribir el archivo
XLSX.writeFileXLSX(wb, 'multiples_hojas.xlsx');
```

Cada hoja puede contener datos independientes. Al abrir el archivo en Excel, verás pestañas “Hoja1”, “Hoja2”, etc. El nombre que des como tercer parámetro será el nombre de la pestaña.

## Ajustar ancho de columnas y alto de filas

Para fijar el ancho de las columnas o la altura de las filas, se usan las propiedades especiales del objeto _worksheet_. En concreto, `ws['!cols']` es un arreglo de objetos donde cada objeto puede tener propiedades como `wch` (anchura en caracteres), `width` o `hidden`. Por ejemplo, para fijar la columna A a 15 caracteres y ocultar la columna B:

```ts
const ws = XLSX.utils.json_to_sheet(this.datos);
ws['!cols'] = [
  { wch: 15 },        // columna A ~15 caracteres
  { hidden: true }    // columna B oculta
];
```

En la documentación se explica que `!cols` almacena metadatos de columnas, incluyendo anchura y visibilidad. Para calcular automáticamente el ancho óptimo, puedes iterar sobre los datos y tomar la longitud máxima de texto, tal como muestra el ejemplo oficial:

```ts
const anchoMax = this.datos
  .reduce((w, r) => Math.max(w, (r.Nombre as string).length), 10);
ws['!cols'] = [{ wch: anchoMax }];
```

De forma similar, `ws['!rows']` permite establecer alturas de filas (objeto con propiedad `hpx` o `hpt` para píxeles o puntos), aunque es menos usado en la práctica.

## Combinar celdas (Merge Cells)

Para combinar (unir) varias celdas en una, se utiliza la propiedad especial `ws['!merges']`, que es un arreglo de rangos. Cada rango tiene la forma `{s:{c:X, r:Y}, e:{c:X2, r:Y2}}` indicando la celda inicial (s) y final (e) de la combinación. Por ejemplo, para combinar el rango A1:B2:

```ts
const ws = XLSX.utils.aoa_to_sheet([
  ["Encabezado1", "Encabezado2"],
  ["Dato1", "Dato2"]
]);
ws['!merges'] = [
  { s: { r: 0, c: 0 }, e: { r: 1, c: 1 } }  // combina A1:B2
];
```

Aquí `r` es el índice de fila (0=A1), `c` es el índice de columna (0=A, 1=B). Esto hace que A1:B2 se fusione en una sola celda. Como indica la documentación, `ws['!merges']` debe contener un arreglo de rangos de fusión. También puedes usar `XLSX.utils.decode_range("A1:B2")` para obtener dicho objeto rango. Por ejemplo:

```ts
ws['!merges'] = [ XLSX.utils.decode_range("A1:B2") ];
```

Al exportar, el contenido de la celda combinada se toma de la esquina superior izquierda (A1 en este caso). Ten en cuenta evitar solapamientos manualmente, ya que no se verifican automáticamente.

## Estilos y formato de celdas

**Importante:** en la versión **gratuita (Community)** de SheetJS no es posible aplicar estilos como colores de fondo, fuentes, bordes, etc. Ese soporte de formato avanzado está reservado para la [versión Pro](https://sheetjs.com/pro) . Para proyectos que requieran mucho formateo visual, existen forks (p.ej. _xlsx-js-style_) o bibliotecas alternativas como [ExcelJS](https://github.com/exceljs/exceljs) que sí admiten estilos. En SheetJS CE puedes definir el tipo de datos de cada celda: el objeto de celda puede llevar propiedades `t` (tipo: `"s"` texto, `"n"` número, `"d"` fecha) y `z` (formato), pero no estilos visuales. Por ejemplo:

```ts
// Crear celda con número y formato de fecha
const ws: XLSX.WorkSheet = { "A1": { t:"n", v: 3.14159 }, "A2": { t:"d", v: new Date(), z: "dd/mm/yyyy" } };
```

Cada objeto de celda puede llevar `t` (type) y `v` (value). La propiedad `z` define el formato numérico (como cadena) si se requiere un formato custom. Sin embargo, propiedades como `font` o `fill` **no están disponibles** en la versión gratuita.

Para completar el formato de archivo Excel (por ejemplo títulos en negrita, color de fondo en celdas, bordes), deberías considerar usar **SheetJS Pro** o bien integrar librerías de terceros mencionadas. Un recurso comunitario menciona forks como `xlsx-js-style` que añaden `ws[cell].s = { /* estilos CSS like */ }` para cada celda, pero esto implica cambiar a esa versión modificada.

## Uso de fórmulas

SheetJS permite insertar fórmulas en las celdas simplemente asignando la propiedad `f` en el objeto de celda. Por ejemplo, para que la celda C1 sea la suma de A1 y B1:

```ts
const ws = XLSX.utils.aoa_to_sheet([
  [1, 2],
  [{ t: "n", v: 3, f: "A1+B1" }]
]);
```

En este objeto, `ws.C1` (índices en 0: fila 1, col 2) tiene `f: "A1+B1"` además de tipo numérico `t:"n"` y valor `v:3`. Al abrir en Excel, se mostrará el resultado de la fórmula. En la documentación de fórmulas se indica que **no** se calculan en JavaScript; es decir, la librería escribe la fórmula en el archivo pero no evalúa su resultado. Si necesitas usar fórmulas complejas y obtener resultados en JavaScript antes de exportar, deberías usar una herramienta dedicada (por ejemplo el complemento de fórmulas de SheetJS Pro).

## Ejemplo completo paso a paso

A continuación un ejemplo integral que reúne varias de las ideas anteriores. Supongamos que tenemos datos en un arreglo de objetos, los queremos exportar con encabezados personalizados, combinación de celdas en el título, y ajustar ancho de columnas:

```ts
import { Component } from '@angular/core';
import * as XLSX from 'xlsx';

interface Usuario { Nombre: string; Edad: number; Ciudad: string; }

@Component({ /* ... */ })
export class MiComponente {
  usuarios: Usuario[] = [
    { Nombre: "Ana", Edad: 28, Ciudad: "Madrid" },
    { Nombre: "Luis", Edad: 35, Ciudad: "Barcelona" },
    { Nombre: "Clara", Edad: 22, Ciudad: "Sevilla" }
  ];

  exportar() {
    // 1) Convertir datos a hoja de cálculo
    const ws: XLSX.WorkSheet = XLSX.utils.json_to_sheet(this.usuarios);
    // 2) Crear libro nuevo y anexar hoja
    const wb: XLSX.WorkBook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, 'Usuarios');
    
    // 3) Añadir encabezados personalizados en la primera fila
    XLSX.utils.sheet_add_aoa(ws, [["Nombre completo", "Edad (años)", "Ciudad"]], { origin: "A1" });
    // 4) Ajustar anchos de columnas
    const maxNombre = Math.max(...this.usuarios.map(u => u.Nombre.length), 10);
    ws['!cols'] = [
      { wch: maxNombre },
      { wch: 10 },
      { wch: 15 }
    ];
    // 5) Combinar la fila de título (por ejemplo, combinar A1:C1 como cabecera general)
    //    (Imaginemos que queremos un gran título que cubra las tres columnas)
    ws['!merges'] = [ XLSX.utils.decode_range("A1:C1") ];
    ws['A1'].v = "Listado de Usuarios";  // ponemos título en A1
    // 6) Escribir el archivo XLSX
    XLSX.writeFileXLSX(wb, 'Usuarios_Detalle.xlsx');
  }
}
```

En este ejemplo:

- Se usa `json_to_sheet` para datos tabulares.
    
- Con `sheet_add_aoa` cambiamos los encabezados (fila 1).
    
- Ajustamos anchos con `!cols`, tomando el nombre más largo para calcularlo.
    
- Creamos un título combinado de fila entera usando `!merges`.
    
- Finalmente exportamos con `writeFileXLSX`.
    

## Resumen de funcionalidades clave

Para facilitar la consulta, a continuación se muestra una breve tabla con las principales herramientas de SheetJS revisadas:

|Funcionalidad|Método/Propiedad|Descripción|
|---|---|---|
|**Crear libro nuevo**|`XLSX.utils.book_new()`|Inicia un workbook vacío.|
|**Agregar hoja**|`XLSX.utils.book_append_sheet(wb, ws, nombre)`|Añade la hoja `ws` al libro `wb` con nombre dado.|
|**Exportar archivo**|`XLSX.writeFileXLSX(wb, nombreArchivo)`|Descarga o guarda el libro en un archivo `.xlsx`.|
|**Tabla HTML → hoja**|`XLSX.utils.table_to_sheet(tablaDOM)`|Convierte una `<table>` DOM a una hoja.|
|**Datos JSON → hoja**|`XLSX.utils.json_to_sheet(arregloObjetos)`|Convierte array de objetos a hoja, keys → encabezados.|
|**Array de arrays → hoja**|`XLSX.utils.aoa_to_sheet(arregloFilas)`|Convierte array de arrays (matriz) a hoja.|
|**Insertar datos a hoja**|`XLSX.utils.sheet_add_aoa(ws, aoa, opts)`|Inserta/actualiza datos en hoja existente (útil para encabezados).|
|**Combinar celdas**|`ws['!merges'] = [rango]`|Define rango(s) combinados en la hoja.|
|**Ancho de columnas**|`ws['!cols'] = [{ wch: ancho }]`|Establece anchura de columnas (wch en caracteres).|
|**Altura de filas**|`ws['!rows'] = [{ hpx: altura }]`|Establece altura de filas (en píxeles).|

## Conclusión

La librería SheetJS (`xlsx`) ofrece muchas funcionalidades para generar Excel de forma programática en Angular. Permite crear libros, agregar hojas, volcar datos desde HTML o JSON, ajustar propiedades avanzadas como anchos y merges, e incluso insertar fórmulas. Las limitaciones principales en la versión libre son la falta de estilos visuales en las celdas (requiere la versión Pro o librerías alternativas), pero para muchos casos de uso es suficiente. La documentación oficial de SheetJS (y sus ejemplos) cubren todos estos casos con mayor detalle. Con estas herramientas, puedes construir archivos Excel complejos directamente desde tu aplicación Angular, dando a los usuarios los archivos descargables con los datos y el formato necesario.

**Referencias:** Documentación oficial de SheetJS (utilidades de conversión, propiedades especiales); ejemplos de uso en Angular; consideraciones de formato. Cada sección arriba enlaza la fuente correspondiente del sitio de SheetJS o foros de desarrollo.