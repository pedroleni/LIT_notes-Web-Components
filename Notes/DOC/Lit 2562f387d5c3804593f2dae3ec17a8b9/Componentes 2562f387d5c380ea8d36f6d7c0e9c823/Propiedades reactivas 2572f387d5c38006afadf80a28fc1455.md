# Propiedades reactivas

Los componentes Lit reciben entrada y almacenan su estado como **campos** o **propiedades** de clase de JavaScript. Las **propiedades reactivas** son propiedades que, al cambiar, pueden **disparar el ciclo de actualización reactivo**, volviendo a renderizar el componente, y opcionalmente **leerse o escribirse** desde/como **atributos**.

```tsx
// JS / TS
class MyElement extends LitElement {
  @property()
  name?: string;
}

```

Lit gestiona tus propiedades reactivas y sus atributos correspondientes. En particular:

- **Actualizaciones reactivas.** Lit genera un **getter/setter** para cada propiedad reactiva. Cuando cambia una propiedad reactiva, el componente programa una actualización.
- **Gestión de atributos.** De forma predeterminada, Lit configura un **atributo observado** correspondiente a la propiedad y actualiza la propiedad cuando el atributo cambia. Los valores de la propiedad también pueden, opcionalmente, **reflejarse** de vuelta al atributo.
- **Propiedades de superclase.** Lit aplica automáticamente las **opciones de propiedad** declaradas por una superclase. No necesitas volver a declararlas salvo que quieras **cambiar opciones**.
- **Actualización del elemento (upgrade).** Si un componente Lit se define **después** de que el elemento ya esté en el DOM, Lit maneja la lógica de **upgrade**, asegurando que cualquier propiedad establecida en el elemento **antes** de su upgrade dispare los efectos reactivos correctos cuando el elemento se actualice.

---

## Propiedades públicas y estado interno

Las **propiedades públicas** forman parte de la **API pública** del componente. En general, las propiedades públicas—especialmente las **reactivas públicas**—deben tratarse como **entrada**.

El componente **no debería** cambiar sus propias propiedades públicas, excepto en respuesta a **entrada del usuario**. Por ejemplo, un componente de menú puede tener una propiedad pública `selected` que el “dueño” del elemento puede inicializar, pero que el propio componente actualiza cuando el usuario selecciona un ítem. En estos casos, el componente debería **despachar un evento** para indicar al dueño del componente que `selected` cambió.

Lit también admite **estado reactivo interno**. El estado reactivo interno se refiere a propiedades reactivas que **no** forman parte de la API del componente. Estas propiedades **no** tienen atributo correspondiente y, en TypeScript, suelen marcarse como `protected` o `private`.

```tsx
// JS / TS
@state()
private _counter = 0;

```

El componente **manipula su propio estado reactivo interno**. En algunos casos, este estado puede **inicializarse a partir de propiedades públicas**—por ejemplo, si hay una transformación costosa entre la propiedad visible para el usuario y el estado interno.

Al igual que con las propiedades reactivas públicas, **actualizar el estado interno** desencadena un ciclo de actualización. Para más información, consulta **Internal reactive state**.

---

## Propiedades reactivas públicas

Declara las propiedades reactivas públicas de tu elemento usando **decoradores** o el **campo estático `properties`**.

En ambos casos, puedes pasar un **objeto de opciones** para configurar características de la propiedad.

### Declarar propiedades con decoradores

Usa el decorador `@property` con una **declaración de campo de clase** para declarar una propiedad reactiva.

```tsx
class MyElement extends LitElement {
  @property({type: String})
  mode?: string;

  @property({attribute: false})
  data = {};
}

```

El argumento de `@property` es un **objeto de opciones**. Omitir el argumento equivale a especificar los **valores por defecto** de todas las opciones.

**Uso de decoradores.** Los decoradores son una característica propuesta de JavaScript, por lo que necesitarás un compilador como **Babel** o el **compilador de TypeScript** para usarlos. Consulta **Enabling decorators** para más detalles.

### Declarar propiedades en el campo estático `properties`

Para declarar propiedades en el campo de clase estático `properties`:

```tsx
class MyElement extends LitElement {
  static properties = {
    mode: {type: String},
    data: {attribute: false},
  };

  constructor() {
    super();
    this.data = {};
  }
}

```

Un objeto de opciones vacío equivale a especificar el **valor por defecto** para todas las opciones.

---

## Evitar problemas con los *class fields* al declarar propiedades

Los **class fields** tienen una interacción problemática con las propiedades reactivas.

Los class fields se definen en la **instancia** del elemento, mientras que las propiedades reactivas se definen como accessors **en el prototipo** del elemento. Según las reglas de JavaScript, una propiedad de instancia **tiene precedencia** y **oculta** a una propiedad del prototipo. Esto significa que los accesores de propiedades reactivas **no funcionan** cuando se usan class fields de forma que al establecer la propiedad **no** se dispare la actualización.

```tsx
class MyElement extends LitElement {
  static properties = { foo: { type: String } }
  foo = 'Default'; // ❌ esto hace que `foo` no sea reactiva
}

```

En **JavaScript**, **no** debes usar class fields al declarar propiedades reactivas. En su lugar, inicializa las propiedades en el **constructor** del elemento:

```tsx
class MyElement extends LitElement {
  static properties = {
    foo: { type: String }
  }
  constructor() {
    super();
    this.foo = 'Default';
  }
}

```

Como alternativa, puedes usar **decoradores estándar con Babel** para declarar propiedades reactivas:

```tsx
class MyElement extends LitElement {
  @property()
  accessor foo = 'Default';
}

```

En **TypeScript**, puedes usar class fields para declarar propiedades reactivas **siempre que** uses uno de estos patrones:

1. **Configurar** `useDefineForClassFields` a `false`.
    
    (Esto ya es la recomendación al usar decoradores con TypeScript).
    

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true, // Si usas decoradores
    "useDefineForClassFields": false}
}

```

```tsx
class MyElement extends LitElement {
  static properties = { foo: { type: String } }
  foo = 'Default';

  @property()
  bar = 'Default';
}

```

1. Añadir la palabra clave **`declare`** en el campo y poner el inicializador en el **constructor**.

```tsx
class MyElement extends LitElement {
  declare foo: string;
  static properties = { foo: { type: String } }
  constructor() {
    super();
    this.foo = 'Default';
  }
}

```

1. Usar la palabra clave **`accessor`** para **auto-accessors**.

```tsx
class MyElement extends LitElement {
  static properties = { foo: { type: String } }
  accessor foo = 'Default';

  @property()
  accessor bar = 'Default';
}

```

---

## Opciones de propiedad

El objeto de opciones puede tener las siguientes propiedades:

- **`attribute`**
    
    Si la propiedad está asociada a un **atributo**, o un **nombre personalizado** para el atributo asociado.
    
    **Predeterminado:** `true`.
    
    Si `attribute` es `false`, se **ignoran** las opciones `converter`, `reflect` y `type`.
    
- **`converter`**
    
    Conversor personalizado para convertir entre **propiedades y atributos**.
    
    Si no se especifica, se usa el **conversor por defecto** de atributos.
    
- **`hasChanged`**
    
    Función que se llama cada vez que se establece la propiedad para determinar si **ha cambiado** y si debe **disparar una actualización**.
    
    Si no se especifica, `LitElement` usa **desigualdad estricta** (`newValue !== oldValue`).
    
- **`noAccessor`**
    
    Establécelo a `true` para evitar que Lit **genere los accesores** por defecto de la propiedad.
    
    Opción rara vez necesaria. **Predeterminado:** `false`.
    
    Más info: *Preventing Lit from generating a property accessor*.
    
- **`reflect`**
    
    Si el valor de la propiedad se **refleja** de vuelta al **atributo** asociado.
    
    **Predeterminado:** `false`.
    
- **`state`**
    
    Establécelo a `true` para declarar la propiedad como **estado reactivo interno**.
    
    El estado interno dispara actualizaciones como las propiedades públicas, pero Lit **no genera un atributo** y los usuarios **no** deberían acceder a él desde fuera del componente.
    
    Equivalente a usar el decorador `@state`. **Predeterminado:** `false`.
    
    Más info: *Internal reactive state*.
    
- **`type`**
    
    Al convertir un **atributo (string)** en una **propiedad**, el conversor por defecto de Lit **parsea** la cadena al **tipo** dado, y viceversa cuando refleja una propiedad a un atributo.
    
    Si `converter` está definido, este campo se **pasa** al conversor.
    
    Si no se especifica `type`, el conversor por defecto lo trata como `type: String`.
    
    Consulta: *Using the default converter*.
    
    > En TypeScript, este campo debería coincidir generalmente con el tipo TS declarado para el campo. Sin embargo, type lo usa el runtime de Lit para serializar/deserializar strings y no es un mecanismo de type-checking.
    > 
    
- **`useDefault`**
    
    Ponlo a `true` para:
    
    - **Evitar** la reflexión inicial del **valor por defecto** cuando `reflect` es `true`.
    - **Restablecer** la propiedad a su **valor por defecto** cuando se **elimina** su atributo correspondiente.
    
    El valor por defecto es el valor inicial de la propiedad **establecido en el constructor** o con un **auto-accessor**. Este valor se **retiene en memoria**, así que es buena práctica **no** usar `useDefault: true` para propiedades **no primitivas** (Object/Array).
    
    Más info: *Enabling attribute reflection* y *Best practices when reflecting attributes*.
    

> Omitir el objeto de opciones o especificarlo vacío equivale a usar los valores predeterminados para todas las opciones.
> 

# Estado reactivo interno

El **estado reactivo interno** se refiere a propiedades reactivas que **no** forman parte de la API pública del componente. Estas propiedades **no** tienen atributos correspondientes y **no** están pensadas para usarse desde fuera del componente. El estado reactivo interno debe ser **gestionado por el propio componente**.

Usa el decorador `@state` para declarar estado reactivo interno:

```tsx
@state()
protected _active = false;

```

Usando el campo estático `properties`, puedes declarar estado reactivo interno con la opción `state: true`:

```tsx
static properties = {
  _active: { state: true }
};

constructor() {
  super();
  this._active = false;
}

```

El estado reactivo interno **no debería referenciarse desde fuera** del componente. En TypeScript, marca estas propiedades como `private` o `protected`. También se recomienda usar una convención como el guion bajo inicial (`_`) para identificar propiedades privadas o protegidas a usuarios de JavaScript.

El estado reactivo interno funciona **igual** que las propiedades reactivas públicas, **excepto** que **no hay atributo** asociado a la propiedad. La **única** opción que puedes especificar para estado reactivo interno es la función `hasChanged`.

El decorador `@state` también puede servir como pista para un **minificador** de código de que el nombre de la propiedad puede cambiarse durante la minificación.

---

# Qué sucede cuando cambian las propiedades

Un cambio en una propiedad puede **disparar un ciclo de actualización reactivo**, lo que hace que el componente **vuelva a renderizar** su plantilla.

Cuando cambia una propiedad, ocurre la siguiente secuencia:

1. Se llama al **setter** de la propiedad.
2. El setter llama a `requestUpdate` del componente.
3. Se **comparan** los valores antiguo y nuevo de la propiedad.
    - Por defecto, Lit usa **desigualdad estricta** (`newValue !== oldValue`).
4. Si la propiedad define `hasChanged`, se llama con los valores antiguo y nuevo.
5. Si se detecta el cambio, se **programa** una actualización **asíncrona**. Si ya hay una programada, solo se ejecuta **una**.
6. Se llama al método `update` del componente, **reflejando** propiedades cambiadas en atributos y **re-renderizando** las plantillas del componente.

Ten en cuenta que si **mutas** (modificas en sitio) una propiedad **objeto** o **array**, **no** disparará una actualización, porque la **referencia** del objeto no ha cambiado. Para más información, consulta **Mutating object and array properties**.

Hay muchas formas de engancharse y modificar el ciclo de actualización reactivo. Para más información, consulta **Reactive update cycle**.

Para más detalles sobre la detección de cambios en propiedades, consulta **Customizing change detection**.

---

# Mutar propiedades objeto y array

Mutar un objeto o array **no cambia su referencia**, por lo que **no** dispara una actualización. Puedes manejar propiedades objeto y array de dos formas:

### 1) Patrón de datos **inmutables**

Trata objetos y arrays como inmutables. Por ejemplo, para eliminar un elemento de `myArray`, construye un **nuevo** array:

```tsx
this.myArray = this.myArray.filter((_, i) => i !== indexToRemove);

```

Aunque este ejemplo es simple, a menudo ayuda usar una librería como **Immer** para gestionar datos inmutables. Esto evita *boilerplate* complejo al actualizar objetos profundamente anidados.

### 2) **Disparar** manualmente una actualización

Muta los datos y llama a `requestUpdate()` para forzar la actualización:

```tsx
this.myArray.splice(indexToRemove, 1);
this.requestUpdate();

```

- Llamado **sin argumentos**, `requestUpdate()` programa una actualización **sin** invocar `hasChanged()`.
- Ojo: `requestUpdate()` **solo** hace que **este componente** se actualice. Si este componente pasa `this.myArray` a un subcomponente, el subcomponente verá que la **referencia** no cambió y **no** se actualizará.

En general, usar **flujo de datos descendente** (top-down) con **objetos inmutables** es lo mejor para la mayoría de aplicaciones: garantiza que cada componente que necesita renderizar un nuevo valor lo haga (y de forma eficiente, ya que las partes del árbol de datos que no cambian no causarán actualizaciones).

Mutar datos directamente y llamar a `requestUpdate()` debe considerarse un **caso avanzado**. En ese caso, tú (u otro sistema) debéis **identificar todos los componentes** que usan los datos mutados y llamar a `requestUpdate()` en cada uno. Cuando esos componentes están repartidos por la aplicación, esto es difícil de gestionar. No hacerlo bien implica que puedes modificar un objeto que se renderiza en **dos sitios** y que solo uno se actualice.

En casos **simples**, cuando sabes que una pieza de datos solo se usa en **un** componente, es seguro mutar los datos y llamar a `requestUpdate()` si lo prefieres.

# Atributos

Las **propiedades** son ideales para recibir datos de JavaScript como entrada, pero los **atributos** son la forma estándar que ofrece HTML para configurar elementos desde el **markup**, sin necesidad de usar JavaScript para establecer propiedades. Proveer **tanto propiedad como atributo** para las propiedades reactivas es clave para que los componentes Lit sean útiles en muchos entornos, incluidos los renderizados sin motor de plantillas en el cliente (por ejemplo, páginas HTML estáticas servidas desde un CMS).

Por defecto, Lit configura un **atributo observado** correspondiente a cada **propiedad reactiva pública**, y **actualiza la propiedad cuando cambia el atributo**. Los valores de la propiedad también pueden, opcionalmente, **reflejarse** (escribirse de vuelta en el atributo).

Mientras que las **propiedades** del elemento pueden ser de **cualquier tipo**, los **atributos** son **siempre cadenas**. Esto afecta a los atributos observados y reflejados de propiedades que no son cadenas:

- Para **observar** un atributo (establecer una propiedad desde un atributo), el valor del atributo debe **convertirse desde string** al **tipo** de la propiedad.
- Para **reflejar** un atributo (establecer un atributo desde una propiedad), el valor de la propiedad debe **convertirse a string**.

Las propiedades booleanas que exponen un atributo deberían **tener por defecto `false`**. Más información en **Boolean attributes**.

---

## Definir el nombre del atributo

Por defecto, Lit crea un **atributo observado** para todas las propiedades reactivas públicas. El **nombre del atributo** observado es el **nombre de la propiedad en minúsculas**:

```tsx
// el atributo observado se llama "myvalue"
@property({ type: Number })
myValue = 0;

```

Para crear un atributo observado con **otro nombre**, usa `attribute` con una **cadena**:

```tsx
// el atributo observado se llamará "my-name"
@property({ attribute: 'my-name' })
myName = 'Ogden';

```

Para **evitar** que se cree un atributo observado para una propiedad, pon `attribute: false`. La propiedad **no** se inicializará desde atributos en el markup y los cambios del atributo **no** le afectarán:

```tsx
// esta propiedad no tendrá atributo observado
@property({ attribute: false })
myData = {};

```

El **estado reactivo interno** **nunca** tiene atributo asociado.

Un atributo observado puede usarse para proporcionar un **valor inicial** a una propiedad desde el markup. Por ejemplo:

```html
<my-element myvalue="99"></my-element>

```

---

## Usar el conversor por defecto

Lit incluye un **conversor por defecto** que maneja los tipos `String`, `Number`, `Boolean`, `Array` y `Object`.

Para usarlo, especifica la opción `type` en la declaración de la propiedad:

```tsx
// Usar el conversor por defecto
@property({ type: Number })
count = 0;

```

Si **no** especificas `type` ni un **conversor personalizado**, el comportamiento es como si hubieras puesto `type: String`.

Tablas de conversión del conversor por defecto:

### De **atributo → propiedad**

| Tipo | Conversión |
| --- | --- |
| **String** | Si el elemento tiene el atributo, la propiedad = valor del atributo. |
| **Number** | Si el elemento tiene el atributo, la propiedad = `Number(attributeValue)`. |
| **Boolean** | Si el elemento **tiene** el atributo ⇒ propiedad = `true`. Si **no** ⇒ `false`. |
| **Object, Array** | Si el elemento tiene el atributo, la propiedad = `JSON.parse(attributeValue)`. |

> Excepto en Boolean, si el elemento no tiene el atributo, la propiedad mantiene su valor por defecto (o undefined si no hay).
> 

### De **propiedad → atributo**

| Tipo | Conversión |
| --- | --- |
| **String, Number** | Si la propiedad está **definida** y **no es null**, el atributo = valor de la propiedad. Si es `null` o `undefined`, **se elimina** el atributo. |
| **Boolean** | Si la propiedad es **truthy**, se **crea** el atributo con **cadena vacía** como valor. Si es **falsy**, **se elimina** el atributo. |
| **Object, Array** | Si la propiedad está **definida** y **no es null**, el atributo = `JSON.stringify(propertyValue)`. Si es `null` o `undefined`, **se elimina**. |

---

## Proveer un conversor personalizado

Puedes especificar un **conversor personalizado** en la declaración de la propiedad con la opción `converter`:

```tsx
myProp: {
  converter: /* conversor personalizado */
}

```

`converter` puede ser **objeto** o **función**. Si es un **objeto**, puede tener las claves `fromAttribute` y `toAttribute`:

```tsx
prop1: {
  converter: {
    fromAttribute: (value, type) => {
      // `value` es string -> conviértelo al tipo `type` y devuélvelo
    },
    toAttribute: (value, type) => {
      // `value` es del tipo `type` -> conviértelo a string y devuélvelo
    }
  }
}

```

Si `converter` es una **función**, se usa en lugar de `fromAttribute`:

```tsx
myProp: {
  converter: (value, type) => {
    // `value` es string -> conviértelo al tipo `type` y devuélvelo
  }
}

```

Si **no** proporcionas `toAttribute` para un atributo **reflejado**, el atributo se establece desde la propiedad usando el **conversor por defecto**.

Si `toAttribute` devuelve `null` o `undefined`, el atributo se **elimina**.

---

## Atributos booleanos

Para que una **propiedad booleana** sea configurable desde un **atributo**, debe **tener por defecto `false`**.

Si por defecto es `true`, **no podrás** ponerla a `false` desde el markup, ya que la **presencia del atributo** (con o sin valor) equivale a `true`. Este es el comportamiento estándar de los atributos en la plataforma web.

Si este comportamiento no encaja con tu caso de uso, tienes opciones:

- Cambia el **nombre de la propiedad** para que su valor por defecto sea `false`. (Ejemplo de la plataforma: el atributo es `disabled` —por defecto `false`—, no `enabled`).
- Usa un **atributo de cadena o número** en su lugar.

---

# Habilitar la reflexión de atributos

Poner `reflect: true` en una propiedad hace que, **cada vez que cambie**, su valor se **refleje** en el **atributo** correspondiente. Los atributos reflejados son útiles para **serializar el estado** del elemento y porque son visibles para **CSS** y para APIs del DOM como `querySelector`.

La opción `useDefault: true` evita que el **valor por defecto** de la propiedad se refleje inicialmente en su atributo. Los **cambios posteriores sí** se reflejan; y si se **elimina** el atributo, la propiedad se **restablece a su valor por defecto**.

Esto coincide con el comportamiento de la plataforma web con atributos como `id`. El valor por defecto de la propiedad `id` de un elemento es `''` (cadena vacía) y, al inicio, el elemento **no** tiene atributo `id`. Si después se establece la propiedad `id` (incluso a cadena vacía), se **refleja** el atributo `id` correspondiente. Si eliminas el atributo `id`, la propiedad vuelve a su valor inicial `''`.

Ejemplos de opciones:

```tsx
// El valor de la propiedad "active" se refleja en el atributo "active"
active: { reflect: true }

// El valor de "variant" se refleja, excepto que el atributo "variant"
// no se establece inicialmente con el valor por defecto de la propiedad.
variant: { reflect: true, useDefault: true }

```

Cuando cambia la propiedad, Lit establece el valor del atributo correspondiente tal como se describe en **Using the default converter** o **Providing a custom converter**.

```tsx
//TS

import { LitElement, html, css, PropertyDeclaration } from 'lit';
import { customElement, property } from 'lit/decorators.js';

@customElement('my-element')
class MyElement extends LitElement {
  // Boolean con reflexión de atributo
  @property({ type: Boolean, reflect: true })
  active: boolean = false;

  // Refleja, pero sin reflejar el valor por defecto inicialmente;
  // y restablece al valor por defecto si se elimina el atributo.
  @property({ reflect: true, useDefault: true } as PropertyDeclaration)
  variant = 'normal';

  render() {
    return html`
      <div>
        <label>
          active:
          <input type="checkbox"
            .value="${this.active}"
            @change="${(e: { target: HTMLInputElement }) =>
              this.active = e.target.checked}">
          ${this.active}
        </label>
      </div>

      <div>
        <label>
          variant:
          <input type="checkbox"
            .value="${this.variant === 'special'}"
            @change="${(e: { target: HTMLInputElement }) =>
              this.variant = e.target.checked ? 'special' : 'normal'}">
          ${this.variant}
        </label>
      </div>
    `;
  }
}

```

```jsx
//JS 

import {__decorate} from 'tslib';
import {LitElement, html, css} from 'lit';

class MyElement extends LitElement {
  static properties = {
    active: {type: Boolean, reflect: true},
  };
  /* playground-fold */
  static styles = css`
    :host {
      display: inline-block; 
      padding: 4px;
    }
    :host([active]) {
      font-weight: 800;
    }
    :host([variant]) {
      outline: 4px solid green;
    }
    :host([variant="special"]) {
      border-radius: 8px; border: 4px solid red;
    }`;

  constructor() {
    super();
    /* playground-fold-end */
    this.active = false;
  }

  variant = 'normal';

  render() {
    return html`
      <div><label>active: <input type="checkbox"
        .value="${this.active}"
        @change="${(e) => (this.active = e.target.checked)}">
        ${this.active}
      </label></div>
      <div><label>variant: <input type="checkbox"
        .value="${this.variant === 'special'}"
        @change="${(e) =>
          (this.variant = e.target.checked ? 'special' : 'normal')}">
        ${this.variant}
      </label></div>
    `;
  }
}
__decorate(
  [property({reflect: true, useDefault: true})],
  MyElement.prototype,
  'variant',
  void 0
);
customElements.define('my-element', MyElement);
```

> Nota práctica: en checkbox lo habitual es enlazar .checked (no .value) para el estado marcado:
> 
> 
> ```html
> <input type="checkbox" .checked=${this.active} @change=${e => this.active = e.target.checked}>
> 
> ```
> 

Lit **rastrea** el estado de la reflexión durante las actualizaciones. Aunque las **propiedades** reflejan a **atributos** y los **atributos** actualizan **propiedades**, evitando teóricamente un bucle infinito, Lit controla cuándo se establecen para **impedir ese bucle**.

# Mejores prácticas al reflejar atributos

Para que los elementos se comporten como se espera y rindan bien, intenta seguir estas buenas prácticas al **reflejar atributos**:

- En general, considera los **atributos como entrada** del elemento desde su “propietario”, más que algo bajo control del propio elemento; por eso, **reflejar propiedades a atributos** debería hacerse **con moderación**. Cuando sea posible, valora usar el **pseudo-selector `:state`** y el **Accessibility Object Model (AOM)**.
- Al reflejar propiedades, normalmente conviene establecer también **`useDefault: true`**, ya que evita que el elemento “genere” espontáneamente atributos que el usuario no ha puesto y ayuda a igualar el comportamiento esperado de la plataforma.
- **No se recomienda** reflejar propiedades de tipo **objeto o array**. Puede hacer que objetos grandes se serialicen al DOM, lo que perjudica el rendimiento y consume memoria extra cuando se usa `useDefault`.
- El decorador **`@property` no altera** los valores asignados a la propiedad reactiva (esto es una buena práctica al usar accesores personalizados). A veces los elementos nativos restringen propiedades a ciertos valores válidos y, si se asigna un valor inválido, la propiedad se resetea a un valor por defecto. **`useDefault: true` no hace esto**: solo restaura el valor por defecto cuando **se elimina el atributo**. Si quieres **modificar/coaccionar** el valor en las asignaciones, **define y decora un *setter* personalizado**.

---

# Accesores de propiedad personalizados

Por defecto, **LitElement** genera un **getter/setter** para todas las propiedades reactivas. El **setter** se invoca siempre que asignas la propiedad:

```jsx
//JS

// Declarar una propiedad
static properties = {
  greeting: {},
};

constructor() {
  super();           // Nota: es `super()`, no `this.super()`
  this.greeting = 'Hello';
}

// Más tarde, asignar la propiedad
this.greeting = 'Hola'; // invoca el accesor generado de 'greeting'

```

```tsx
//TS

@property()
greeting: string = 'Hello';
...
// Later, set the property
this.greeting = 'Hola'; // invokes greeting's generated property accessor

```

Los accesores generados llaman automáticamente a `requestUpdate()`, iniciando una actualización si aún no hay una en curso.

## Crear accesores personalizados

Para especificar cómo funciona el get/set de una propiedad, puedes definir tu **propio par getter/setter**. Por ejemplo:

```jsx

///JS
static properties = {
  prop: {},
};

_prop = 0;

set prop(val: number) {
  this._prop = Math.floor(val);
}

get prop() {
  return this._prop;
}

```

```jsx

///TS
private _prop = 0;

@property()
set prop(val: number) {
  this._prop = Math.floor(val);
}

get prop() { return this._prop; }

```

Para usar accesores personalizados con los decoradores `@property` o `@state`, **coloca el decorador en el *setter*** (como arriba). Los *setters* decorados con `@property` o `@state` llaman a **`requestUpdate()`**.

En la mayoría de los casos **no necesitas** crear accesores personalizados. Para **calcular valores a partir de propiedades existentes**, se recomienda usar el **callback `willUpdate`**, que te permite establecer valores **durante** el ciclo de actualización **sin** disparar otra actualización adicional. Para realizar una acción personalizada **después** de que el elemento se actualice, usa el **callback `updated`**. Un *setter* personalizado se reserva para casos raros en los que es importante **validar sincrónicamente** cualquier valor que el usuario asigne.

Si tu clase **define sus propios accesores** para una propiedad, Lit **no los sobrescribe**. Si tu clase **no** define accesores para una propiedad, Lit **los generará**, incluso si una **superclase** definió la propiedad o sus accesores.

---

# Evitar que Lit genere un accesor de propiedad

En casos raros, una subclase puede necesitar **cambiar o añadir opciones** de una propiedad que existe en su superclase.

Para **evitar** que Lit genere un accesor que sobrescriba el accesor definido en la superclase, pon **`noAccessor: true`** en la declaración de la propiedad:

```tsx
static properties = {
  myProp: { type: Number, noAccessor: true }
};

```

Si **defines tus propios accesores**, **no** necesitas establecer `noAccessor`.

---

# Personalizar la detección de cambios

Todas las propiedades reactivas tienen una función **`hasChanged()`** que se llama cuando se establece la propiedad.

`hasChanged` compara los **valores antiguo y nuevo** y determina si la propiedad **ha cambiado**. Si `hasChanged()` devuelve `true`, Lit inicia una **actualización** del elemento si aún no hay una programada. (Para más detalles sobre actualizaciones, ver **Reactive update cycle**).

La implementación por defecto usa **desigualdad estricta**: `hasChanged()` devuelve `true` si `newVal !== oldVal`.

Para personalizar `hasChanged()` en una propiedad, especifícalo como **opción** de la propiedad:

```tsx
//JS

static properties = {
  myProp: {
    hasChanged(newVal: string, oldVal: string) {
      return newVal?.toLowerCase() !== oldVal?.toLowerCase();
    }
  }
};

```

```tsx
//TS
@property({
  hasChanged(newVal: string, oldVal: string) {
    return newVal?.toLowerCase() !== oldVal?.toLowerCase();
  }
})
myProp: string | undefined;

```

En el siguiente ejemplo, `hasChanged()` solo devuelve `true` para **valores impares**:

```tsx
//JS

import { LitElement, html } from 'lit';

class MyElement extends LitElement {
  static properties = {
    value: {
      // solo actualizar para valores impares
      hasChanged(newVal: number, oldVal: number) {
        const changed = newVal % 2 === 1;
        console.log(`${newVal}, ${oldVal}, ${changed}`);
        return changed;
      },
    },
  };

  constructor() {
    super();
    this.value = 1;
  }

  render() {
    return html`
      <p>${this.value}</p>
      <button @click=${this.getNewVal}>Get new value</button>
    `;
  }

  getNewVal() {
    this.value = Math.floor(Math.random() * 100);
  }
}
customElements.define('my-element', MyElement);

```

```tsx
//TS

import {LitElement, html} from 'lit';
import {customElement, property} from 'lit/decorators.js';

@customElement('my-element')
class MyElement extends LitElement {
  @property({
    // only update for odd values of newVal.
    hasChanged(newVal: number, oldVal: number) {
      const hasChanged: boolean = newVal % 2 == 1;
      console.log(`${newVal}, ${oldVal}, ${hasChanged}`);
      return hasChanged;
    },
  })
  value: number = 1;

  render() {
    return html`
      <p>${this.value}</p>
      <button @click="${this.getNewVal}">Get new value</button>
    `;
  }

  getNewVal() {
    this.value = Math.floor(Math.random() * 100);
  }
}
```