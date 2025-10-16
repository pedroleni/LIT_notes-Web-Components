# Renderizado

Agrega una **plantilla** (`template`) a tu componente para definir qué debe mostrar.

Las plantillas pueden incluir **expresiones**, que son marcadores para contenido dinámico.

Para definir una plantilla en un componente Lit, añade un método `render()`:

```tsx
import { LitElement, html } from 'lit';

class MyElement extends LitElement {
  render() {
    return html`<p>Hola desde mi plantilla.</p>`;
  }
}
customElements.define('my-element', MyElement);

```

Las plantillas se escriben en **HTML dentro de un template literal** de JavaScript, usando la función `html` de Lit.

👉 Dentro de las plantillas se pueden incluir **expresiones de JavaScript**:

- Texto dinámico
- Atributos
- Propiedades
- Event listeners

El método `render()` también puede contener JavaScript adicional (ejemplo: crear variables locales y usarlas en las expresiones).

```tsx

// ejemplo variables en el render 
render() {
    const {remaining, running} = this;
    const min = Math.floor(remaining / 60000);
    const sec = pad(min, Math.floor(remaining / 1000 % 60));
    const hun = pad(true, Math.floor(remaining % 1000 / 10));
    return html`
      ${min ? `${min}:${sec}` : `${sec}.${hun}`}
      <footer>
        ${remaining === 0 ? '' : running ?
          html`<span @click=${this.pause}>${pause}</span>` :
          html`<span @click=${this.start}>${play}</span>`}
        <span @click=${this.reset}>${replay}</span>
      </footer>
    `;
  }
```

---

## Valores renderizables

Normalmente, `render()` devuelve un objeto `TemplateResult` (lo que devuelve la función `html`).

Pero en realidad puede devolver cualquier cosa que Lit sepa renderizar como hijo de un elemento HTML:

- Valores primitivos: `string`, `number`, `boolean`.
- Objetos `TemplateResult` creados con `html`.

```tsx
import { LitElement, html } from 'lit';

class MyTemplateResultExample extends LitElement {
  render() {
    const chip = html`<span style="border:1px solid;padding:2px 6px;border-radius:8px;">Chip</span>`;
    return html`<div>Composición de plantillas: ${chip}</div>`;
  }
}
customElements.define('my-template-result-example', MyTemplateResultExample);

```

- Nodos del DOM.

```tsx
import { LitElement, html } from 'lit';

class MyDomNodeExample extends LitElement {
  render() {
    const node = document.createElement('strong');
    node.textContent = 'Nodo creado a mano';
    return html`<p>Incluyendo un nodo: ${node}</p>`;
  }
}
customElements.define('my-dom-node-example', MyDomNodeExample);

```

- Valores especiales `nothing` y `noChange`.

## `nothing`

- **Qué hace:** “no pongas nada”.
- **Hijos (contenido):** vacía esa zona del DOM. Si antes había nodos, los quita.
- **Atributos:** elimina el atributo (no lo deja como `""`, lo quita).
- **Propiedades/eventos:** no escribe nada ahí (efecto práctico: no cambia lo que hubiese).

**Ejemplos**

```tsx
html`${cond ? html`<span>OK</span>` : nothing}`          // quita/pon el <span>
html`<input title=${cond ? 'Info' : nothing}>`           // quita el atributo title

```

## `noChange`

- **Qué hace:** “no toques lo que ya hay”.
- **Hijos (contenido):** mantiene los nodos existentes sin re-renderizarlos.
- **Atributos/propiedades/eventos:** no actualiza ese binding; se queda como estaba.
- **Uso típico:** en directivas o cuando quieres saltarte una actualización por rendimiento.
- **Nota:** en el primer render (si no había valor previo), **equivale de facto a no poner nada**.

**Ejemplo**

```tsx
html`${deberiaActualizar ? html`<span>nuevo</span>` : noChange}`  // conserva el contenido anterior

```

### Resumen rápido

- `nothing` ⇒ **vacía/omite**.
- `noChange` ⇒ **conserva/salta actualización**.

- Arrays o iterables con cualquiera de los tipos anteriores.

```tsx
import { LitElement, html } from "lit";
import { customElement, property } from "lit/decorators.js";

@customElement("list-map-demo")
export class ListMapDemo extends LitElement {
  @property() items: string[] = ["🍎", "🍐", "🍊"];

  render() {
    return html`
      <ul>
        ${this.items.map((item) => html`<li>${item}</li>`)}
      </ul>
    `;
  }
}

```

🔹 Diferencia: en las expresiones hijas (`child expressions`) también se puede renderizar un `SVGTemplateResult` (devuelto por la función `svg`), pero solo dentro de un `<svg>`.

---

## Escribir un buen método `render()`

Para aprovechar al máximo el modelo de renderizado funcional de Lit, el método `render()` debería:

✔️ **Evitar cambiar el estado** del componente.

✔️ **Evitar efectos secundarios** (side effects).

✔️ Usar únicamente **propiedades del componente** como entrada.

✔️ Ser **determinista**: mismo input → mismo resultado.

De esta forma, la plantilla es predecible y fácil de razonar.

👉 Regla de oro: evita manipular el DOM directamente fuera de `render()`.

En su lugar, expresa la UI como función del estado y captura ese estado en **propiedades reactivas**.

Ejemplo: si quieres actualizar la UI tras un evento, haz que el event listener actualice una propiedad reactiva → `render()` se encargará del DOM.

---

## Composición de plantillas

Puedes **componer plantillas** a partir de otras plantillas.

Ejemplo: componente `<my-page>` que compone cabecera, artículo y pie:

```jsx
import { LitElement, html } from 'lit';

class MyPage extends LitElement {
  static properties = {
    article: { attribute: false },
  };

  constructor() {
    super();
    this.article = {
      title: 'Mi artículo interesante',
      text: 'Un texto ingenioso.',
    };
  }

  headerTemplate() {
    return html`<header>${this.article.title}</header>`;
  }

  articleTemplate() {
    return html`<article>${this.article.text}</article>`;
  }

  footerTemplate() {
    return html`<footer>Tu pie de página aquí.</footer>`;
  }

  render() {
    return html`
      ${this.headerTemplate()}
      ${this.articleTemplate()}
      ${this.footerTemplate()}
    `;
  }
}
customElements.define('my-page', MyPage);

```

```tsx
import {LitElement, html} from 'lit';
import {customElement, property} from 'lit/decorators.js';

@customElement('my-page')
class MyPage extends LitElement {

  @property({attribute: false})
  article = {
    title: 'My Nifty Article',
    text: 'Some witty text.',
  };

  headerTemplate() {
    return html`<header>${this.article.title}</header>`;
  }

  articleTemplate() {
    return html`<article>${this.article.text}</article>`;
  }

  footerTemplate() {
    return html`<footer>Your footer here.</footer>`;
  }

  render() {
    return html`
      ${this.headerTemplate()}
      ${this.articleTemplate()}
      ${this.footerTemplate()}
    `;
  }
}

```

➡️ Aquí cada subplantilla es un método, lo que permite a una subclase sobreescribir partes específicas.

Otra forma es **importar otros elementos** y usarlos en la plantilla:

```tsx
// my-page.ts

import { LitElement, html } from 'lit';

import './my-header.js';
import './my-article.js';
import './my-footer.js';

class MyPage extends LitElement {
  render() {
    return html`
      <my-header></my-header>
      <my-article></my-article>
      <my-footer></my-footer>
    `;
  }
}
customElements.define('my-page', MyPage);

```

```tsx
// my-header.ts
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';

@customElement('my-header')
class MyHeader extends LitElement {
  render() {
    return html`
      <header>header</header>
    `;
  }
}

```

```tsx
// my-article.ts
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';

@customElement('my-article')
class MyArticle extends LitElement {
  render() {
    return html`
      <article>article</article>
    `;
  }
}

```

```tsx
// my-footer.ts
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';

@customElement('my-footer')
class MyFooter extends LitElement {
  render() {
    return html`
      <footer>footer</footer>
    `;
  }
}

```

```html
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="./my-page.js" type="module"></script>
  <title>lit-element code sample</title>
</head>
  <body>
    <my-page></my-page>
  </body>
</html>

```

---

## Cuándo se renderizan las plantillas

- Un componente Lit renderiza su plantilla **cuando se añade al DOM**.
- Después, **cualquier cambio en propiedades reactivas** inicia un ciclo de actualización.
- Lit **agrupa actualizaciones** para ser más eficiente (varios cambios → un único render).
- Solo se re-renderizan las partes que cambian, no toda la plantilla.

Esto es posible porque Lit **parsea el HTML una sola vez** y luego solo sustituye los valores dinámicos.

---

## Encapsulación del DOM

Lit usa **shadow DOM** para encapsular el contenido renderizado.

El shadow DOM crea un árbol DOM aislado del documento principal.

Ventajas:

- Aislamiento de estilos.
- Interoperabilidad entre componentes.
- Encapsulación clara del componente.

---