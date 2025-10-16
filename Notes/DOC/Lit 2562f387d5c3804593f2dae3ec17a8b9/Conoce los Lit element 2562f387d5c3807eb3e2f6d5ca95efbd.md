# Conoce los Lit element

### ¿Qué puedo construir con Lit?

¡Con Lit puedes construir prácticamente cualquier tipo de interfaz web!

Lo primero que debes saber es que **cada componente de Lit es un componente web estándar**.

Los componentes web tienen la superpotencia de la interoperabilidad: al estar soportados de forma nativa por los navegadores, se pueden usar en cualquier entorno HTML, con cualquier framework o incluso sin ninguno.

Esto convierte a Lit en una opción ideal para desarrollar **componentes reutilizables o sistemas de diseño**. Los componentes creados con Lit pueden usarse en múltiples aplicaciones y sitios, aunque estén construidos con diferentes stacks de front-end.

Los desarrolladores que usen esos componentes no necesitan escribir ni ver código de Lit: simplemente los utilizan igual que lo harían con cualquier elemento HTML nativo.

Lit también es perfecto para **mejorar progresivamente** sitios HTML básicos. Los navegadores reconocerán los componentes de Lit en tu marcado y los inicializarán automáticamente, ya sea que tu sitio esté hecho a mano, gestionado con un CMS, construido con un framework del lado del servidor o generado con herramientas como Jekyll o Eleventy.

Por supuesto, también puedes crear **aplicaciones muy interactivas y con muchas funcionalidades** usando componentes de Lit, tal y como harías con frameworks como React o Vue. Las capacidades y la experiencia de desarrollo de Lit son comparables a estas alternativas populares, pero Lit **reduce la dependencia de una sola herramienta, maximiza la flexibilidad y fomenta la mantenibilidad** al apoyarse en el modelo de componentes nativo del navegador.

Cuando construyes una app con Lit, es fácil integrar **componentes web “vanilla”** o creados con otras librerías. Incluso puedes actualizar a una nueva versión mayor de Lit —o migrar hacia otra librería— **componente a componente**, sin interrumpir el desarrollo del producto.

---

### ¿Cómo es desarrollar con Lit?

Si ya has trabajado con desarrollo web moderno basado en componentes, te sentirás como en casa con Lit.

Y aunque nunca hayas desarrollado con componentes, pensamos que encontrarás Lit muy accesible.

Cada componente de Lit es una **unidad de interfaz autocontenida**, ensamblada a partir de bloques más pequeños: elementos HTML estándar y otros componentes web. A su vez, cada componente de Lit puede convertirse en un bloque de construcción que se use —en un documento HTML, en otro componente web o en un componente de framework— para armar interfaces más grandes y complejas.

Aquí tienes un ejemplo pequeño, pero no trivial, de un componente (un temporizador de cuenta regresiva) que ilustra cómo se ve el código de Lit y resalta varias de sus características principales:

---

TS 

```tsx
import {LitElement, html, css} from 'lit';
import {customElement, property, state} from 'lit/decorators.js';
/* playground-fold */
import {play, pause, replay} from './icons.js';
/* playground-fold-end */

@customElement("my-timer")
export class MyTimer extends LitElement {
  static styles = css`/* playground-fold */

    :host {
      display: inline-block;
      min-width: 4em;
      text-align: center;
      padding: 0.2em;
      margin: 0.2em 0.1em;
    }
    footer {
      user-select: none;
      font-size: 0.6em;
    }
    /* playground-fold-end */`;

  @property() duration = 60;
  @state() private end: number | null = null;
  @state() private remaining = 0;

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
  /* playground-fold */

  start() {
    this.end = Date.now() + this.remaining;
    this.tick();
  }

  pause() {
    this.end = null;
  }

  reset() {
    const running = this.running;
    this.remaining = this.duration * 1000;
    this.end = running ? Date.now() + this.remaining : null;
  }

  tick() {
    if (this.running) {
      this.remaining = Math.max(0, this.end! - Date.now());
      requestAnimationFrame(() => this.tick());
    }
  }

  get running() {
    return this.end && this.remaining;
  }

  connectedCallback() {
    super.connectedCallback();
    this.reset();
  }/* playground-fold-end */

}
/* playground-fold */

function pad(pad: unknown, val: number) {
  return pad ? String(val).padStart(2, '0') : val;
}/* playground-fold-end */

```

```tsx
<!doctype html>
<head><!-- playground-fold -->
  <link rel="preconnect" href="https://fonts.gstatic.com">
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@1,800&display=swap" rel="stylesheet">
  <script type="module" src="./my-timer.js"></script>
  <style>
    body {
      font-family: 'JetBrains Mono', monospace;
      font-size: 36px;
    }
  </style>
  <!-- playground-fold-end -->
</head>
<body>
  <my-timer duration="7"></my-timer>
  <my-timer duration="60"></my-timer>
  <my-timer duration="300"></my-timer>
</body>

```

```tsx
import {html} from 'lit';

export const replay = html`<svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Replay</title><g><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/></g><g><g/><path d="M12,5V1L7,6l5,5V7c3.31,0,6,2.69,6,6s-2.69,6-6,6s-6-2.69-6-6H4c0,4.42,3.58,8,8,8s8-3.58,8-8S16.42,5,12,5z"/></g></svg>`;
export const pause = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Pause</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"/></svg>`;
export const play = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Play</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M10 8.64L15.27 12 10 15.36V8.64M8 5v14l11-7L8 5z"/></svg>`;

```

---

JS

```jsx
import {LitElement, html, css} from 'lit';
/* playground-fold */
import {play, pause, replay} from './icons.js';
/* playground-fold-end */

export class MyTimer extends LitElement {
  static properties = {
    duration: {},
    end: {state: true},
    remaining: {state: true},
  };
  static styles = css`/* playground-fold */

    :host {
      display: inline-block;
      min-width: 4em;
      text-align: center;
      padding: 0.2em;
      margin: 0.2em 0.1em;
    }
    footer {
      user-select: none;
      font-size: 0.6em;
    }
    /* playground-fold-end */`;

  constructor() {
    super();
    this.duration = 60;
    this.end = null;
    this.remaining = 0;
  }

  render() {
    const {remaining, running} = this;
    const min = Math.floor(remaining / 60000);
    const sec = pad(min, Math.floor((remaining / 1000) % 60));
    const hun = pad(true, Math.floor((remaining % 1000) / 10));
    return html`
      ${min ? `${min}:${sec}` : `${sec}.${hun}`}
      <footer>
        ${
          remaining === 0
            ? ''
            : running
            ? html`<span @click=${this.pause}>${pause}</span>`
            : html`<span @click=${this.start}>${play}</span>`
        }
        <span @click=${this.reset}>${replay}</span>
      </footer>
    `;
  }
  /* playground-fold */

  start() {
    this.end = Date.now() + this.remaining;
    this.tick();
  }

  pause() {
    this.end = null;
  }

  reset() {
    const running = this.running;
    this.remaining = this.duration * 1000;
    this.end = running ? Date.now() + this.remaining : null;
  }

  tick() {
    if (this.running) {
      this.remaining = Math.max(0, this.end - Date.now());
      requestAnimationFrame(() => this.tick());
    }
  }

  get running() {
    return this.end && this.remaining;
  }

  connectedCallback() {
    super.connectedCallback();
    this.reset();
  } /* playground-fold-end */
}
customElements.define('my-timer', MyTimer);
/* playground-fold */

function pad(pad, val) {
  return pad ? String(val).padStart(2, '0') : val;
} /* playground-fold-end */

```

```html
<!doctype html>
<head><!-- playground-fold -->
  <link rel="preconnect" href="https://fonts.gstatic.com">
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@1,800&display=swap" rel="stylesheet">
  <script type="module" src="./my-timer.js"></script>
  <style>
    body {
      font-family: 'JetBrains Mono', monospace;
      font-size: 36px;
    }
  </style>
  <!-- playground-fold-end -->
</head>
<body>
  <my-timer duration="7"></my-timer>
  <my-timer duration="60"></my-timer>
  <my-timer duration="300"></my-timer>
</body>

```

```jsx
import {html} from 'lit';

export const replay = html`<svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Replay</title><g><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/></g><g><g/><path d="M12,5V1L7,6l5,5V7c3.31,0,6,2.69,6,6s-2.69,6-6,6s-6-2.69-6-6H4c0,4.42,3.58,8,8,8s8-3.58,8-8S16.42,5,12,5z"/></g></svg>`;
export const pause = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Pause</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"/></svg>`;
export const play = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Play</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M10 8.64L15.27 12 10 15.36V8.64M8 5v14l11-7L8 5z"/></svg>`;

```

### 🔹 Algunas cosas a tener en cuenta:

- La **característica principal de Lit** es la clase base `LitElement`, una extensión práctica y versátil de `HTMLElement`. Para definir tus propios componentes, simplemente heredas de ella.
- Las **plantillas declarativas y expresivas de Lit** (basadas en *tagged template literals* de JavaScript) facilitan describir cómo debe renderizarse un componente.
- Las **propiedades reactivas** representan la API pública de un componente y/o su estado interno; tu componente se vuelve a renderizar automáticamente cuando una propiedad reactiva cambia.
- Los **estilos están encapsulados por defecto**, lo que mantiene simples tus selectores CSS y asegura que el estilo de tu componente no contamine (ni sea contaminado por) el contexto que lo rodea.
- Lit funciona perfectamente con **JavaScript puro**, o con **TypeScript** para una experiencia aún mejor gracias a los decoradores y las declaraciones de tipos.
- Lit **no requiere compilación ni build en desarrollo**, por lo que puedes usarlo prácticamente sin herramientas adicionales si lo prefieres. Además, cuenta con soporte de primera clase en IDEs (autocompletado, linting, etc.) y con herramientas disponibles para producción (localización, minificación de plantillas, etc.).

---

### 🔹 ¿Por qué debería elegir Lit?

Como ya se mencionó, Lit es una excelente elección para construir todo tipo de interfaces web, combinando las ventajas de la **interoperabilidad de los Web Components** con una **experiencia de desarrollo moderna y ergonómica**.

Lit además es:

- **Simple.** Construido sobre los estándares de Web Components, Lit añade solo lo necesario para que seas productivo y trabajes cómodo: reactividad, plantillas declarativas y un puñado de funcionalidades diseñadas para reducir código repetitivo.
- **Rápido.** Las actualizaciones son veloces porque Lit rastrea las partes dinámicas de tu UI y actualiza solo esas cuando cambia el estado subyacente. No necesita reconstruir todo un árbol virtual ni compararlo con el DOM actual.
- **Ligero.** Con un peso de unos **5 KB** (minificado y comprimido), Lit ayuda a mantener pequeños los *bundles* y cortos los tiempos de carga.
- **Respaldado por expertos.** El equipo detrás de Lit ha estado involucrado en los Web Components desde el inicio. Colaboran en Google manteniendo decenas de miles de componentes, ofrecen un conjunto completo de polyfills y participan activamente en la evolución de estándares y la comunidad.

Cada característica de Lit está diseñada cuidadosamente pensando en la **evolución de la plataforma web**: la idea es que aproveches al máximo lo que hoy ofrece el navegador mientras escribes código preparado para beneficiarse de las mejoras futuras.