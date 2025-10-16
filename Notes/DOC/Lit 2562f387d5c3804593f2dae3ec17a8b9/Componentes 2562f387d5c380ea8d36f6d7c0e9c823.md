# Componentes

## Visión general de los componentes

Un **componente Lit** es una pieza reutilizable de interfaz de usuario (UI).

Puedes pensar en un componente Lit como un **contenedor** que tiene un estado y muestra una UI basada en ese estado.

- Puede reaccionar a la interacción del usuario.
- Puede lanzar eventos.
- Hace lo que esperarías de cualquier componente de UI.
- Y además, un componente Lit **es un elemento HTML**, así que tiene todas las APIs estándar de los elementos nativos.

---

### Conceptos clave al crear un componente Lit:

- **Definir un componente**
    
    Un componente Lit se implementa como un **custom element** (elemento personalizado) registrado en el navegador.
    
- **Renderizado**
    
    Cada componente tiene un método `render()` que se llama para **pintar el contenido** del componente.
    
    En este método defines la **plantilla (template)** del componente.
    
- **Propiedades reactivas**
    
    Las propiedades almacenan el **estado** del componente.
    
    Cuando cambias una o más de estas propiedades, se dispara un **ciclo de actualización**, y el componente se vuelve a renderizar automáticamente.
    
- **Estilos**
    
    Un componente puede definir **estilos encapsulados** para controlar su propia apariencia.
    
- **Ciclo de vida (lifecycle)**
    
    Lit define una serie de **callbacks** que puedes sobrescribir para engancharte al ciclo de vida del componente.
    
    Ejemplo: ejecutar código cuando el elemento se añade al DOM, o cada vez que se actualiza.
    

## Ejemplo de componente simple

`📒`TS 

```tsx
import { LitElement, css, html } from 'lit';
import { customElement, property } from 'lit/decorators.js';

// Definimos un nuevo elemento personalizado <simple-greeting>
@customElement('simple-greeting')
export class SimpleGreeting extends LitElement {
  
  // Estilos propios del componente (encapsulados)
  static styles = css`
    :host {
      color: blue;
    }
  `;

  // Propiedad reactiva
  @property()
  name?: string = 'World';

  // Renderizado basado en el estado (propiedades)
  render() {
    return html`<p>Hello, ${this.name}!</p>`;
  }
}

```

📒JS 

```jsx
import {LitElement, css, html} from 'lit';

export class SimpleGreeting extends LitElement {
  static properties = {
    name: {},
  };
  // Define scoped styles right with your component, in plain CSS
  static styles = css`
    :host {
      color: blue;
    }
  `;

  constructor() {
    super();
    // Declare reactive properties
    this.name = 'World';
  }

  // Render the UI as a function of component state
  render() {
    return html`<p>Hello, ${this.name}!</p>`;
  }
}
customElements.define('simple-greeting', SimpleGreeting);
```

[Definir un componente](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Definir%20un%20componente%202572f387d5c380758b82caa473012138.md)

[Renderizado](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Renderizado%202572f387d5c380e7b020c2fc6b7c87d2.md)

[Propiedades reactivas](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Propiedades%20reactivas%202572f387d5c38006afadf80a28fc1455.md)

[Styles](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Styles%202572f387d5c3808d8e4dca520c3af890.md)

[Ciclo de vida](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Ciclo%20de%20vida%202572f387d5c380069ec8c2249985f85e.md)

[](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Sin%20t%C3%ADtulo%202572f387d5c3805a9176c9241fd3e783.md)

[](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Sin%20t%C3%ADtulo%202572f387d5c380288102cd9a42bcfc14.md)

[](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Sin%20t%C3%ADtulo%202572f387d5c3804abcf6d7bb7defa79e.md)

[](Componentes%202562f387d5c380ea8d36f6d7c0e9c823/Sin%20t%C3%ADtulo%202572f387d5c380738430c4bb63bd1bc6.md)