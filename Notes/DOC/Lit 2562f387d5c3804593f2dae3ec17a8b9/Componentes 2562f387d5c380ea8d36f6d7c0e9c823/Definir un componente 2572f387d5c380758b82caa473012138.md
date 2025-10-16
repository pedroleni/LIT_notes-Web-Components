# Definir un componente

Defines un componente de Lit creando una **clase que extiende `LitElement`** y registrando esa clase en el navegador:

```tsx
@customElement('simple-greeting')
export class SimpleGreeting extends LitElement { /* ... */ }

```

El decorador `@customElement` es una forma abreviada de llamar a `customElements.define`.

Ese método registra la clase del componente como **custom element** en el navegador y la asocia con un nombre de etiqueta (en este caso, `simple-greeting`).

Si usas **JavaScript** o no utilizas decoradores, puedes registrar el componente directamente:

```tsx
export class SimpleGreeting extends LitElement { /* ... */ }
customElements.define('simple-greeting', SimpleGreeting);

```

---

## Un componente Lit es un elemento HTML

Cuando defines un componente Lit, en realidad estás definiendo un **nuevo elemento HTML personalizado**.

Puedes usarlo igual que usarías cualquier elemento nativo:

```html
<simple-greeting name="Markup"></simple-greeting>

```

O bien crearlo por JavaScript:

```jsx
const greeting = document.createElement('simple-greeting');

```

La clase base `LitElement` hereda de `HTMLElement`, por lo que un componente Lit tiene **todas las propiedades y métodos estándar** de cualquier elemento HTML.

Más en detalle:

- `LitElement` hereda de **ReactiveElement**, que implementa propiedades reactivas.
- `ReactiveElement` hereda de **HTMLElement**, la interfaz estándar del DOM para todos los elementos HTML nativos y personalizados.

📌 En resumen:

- **HTMLElement** → funcionalidad estándar del DOM.
- **ReactiveElement** → gestión de propiedades y atributos reactivos.
- **LitElement** → renderizado con plantillas (`render()`).

![image.png](Definir%20un%20componente%202572f387d5c380758b82caa473012138/image.png)

---

## Buenas prácticas con TypeScript

TypeScript es capaz de **inferir automáticamente** el tipo de un elemento HTML creado según el nombre de la etiqueta.

Ejemplo:

```tsx
document.createElement('img');
// Devuelve HTMLImageElement con su propiedad src: string

```

Podemos hacer lo mismo con **custom elements**, añadiendo una entrada al mapa `HTMLElementTagNameMap`.

Ejemplo en Lit:

```tsx
@customElement('my-element')
export class MyElement extends LitElement {
  @property({ type: Number })
  aNumber: number = 5;
  /* ... */
}

declare global {
  interface HTMLElementTagNameMap {
    "my-element": MyElement;
  }
}

```

De esta forma, TypeScript entiende que cuando hacemos:

```tsx
const myElement = document.createElement('my-element');
myElement.aNumber = 10; // ✅ correcto, TS reconoce la propiedad

```

👉 Recomendación oficial:

- Añadir siempre una entrada en `HTMLElementTagNameMap` para cada componente creado con TypeScript.
- Publicar los **typings (`.d.ts`)** en tu paquete npm, para que otros desarrolladores también tengan el tipado correcto.

---

¿Quieres que te prepare un **resumen gráfico** con un diagrama de herencia de LitElement → ReactiveElement → HTMLElement y un ejemplo de tipado en TS en formato tabla?