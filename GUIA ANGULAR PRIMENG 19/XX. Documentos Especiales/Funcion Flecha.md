Las **funciones flecha** (arrow functions) son una forma más concisa de definir funciones en JavaScript y TypeScript. 

---

## 1. Sintaxis básica

```ts
// Función tradicional
function suma(a: number, b: number): number {
  return a + b;
}

// Función flecha equivalente
const suma = (a: number, b: number): number => {
  return a + b;
};
```

- Se eliminan la palabra clave `function` y, si la llamas anónima, desaparece el nombre.
    
- Los parámetros van entre paréntesis: `(a, b)`.
    
- Después de los parámetros, va el operador `=>`.
    
- El bloque entre `{}` contiene el cuerpo de la función.
    

---

## 2. Retorno implícito

Si el cuerpo de la función flecha es **una sola expresión**, puedes omitir las llaves `{}` y la palabra `return`. Ese valor se devuelve automáticamente:

```ts
// Con llaves y return explícito
const doble = (x: number): number => {
  return x * 2;
};

// Retorno implícito (más compacto)
const doble = (x: number): number => x * 2;
```

---

## 3. Parámetros

- **Un solo parámetro**: puedes omitir los paréntesis.
    
    ```ts
    const cuadrado = x => x * x;
    ```
    
- **Sin parámetros**: debes usar paréntesis vacíos.
    
    ```ts
    const tiempoActual = () => new Date();
    ```
    

---

## 6. Usos típicos

- **Callbacks** en métodos de arrays:
    
    ```ts
    nums = [1, 2, 3];
    const alCuadrado = nums.map(n => n * n);
    
// alCuadrado(n : number) {
//	return n*n
// }
    ```
    
- **Encadenamiento de promesas**:
    
    ```ts
    fetch(url)
      .then(res => res.json())
      .then(data => console.log(data));
    ```
    
---

## 7. Resumen de diferencias clave

| Característica    | Función tradicional  | Arrow function |
| ----------------- | -------------------- | -------------- |
| Sintaxis          | `function foo() {…}` | `() => {…}`    |
| Retorno implícito | No                   | Sí (sin `{}`)  |
