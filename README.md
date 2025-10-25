# Aprendiendo Vue

## 1. Ordenando el proyecto: Arquitectura

### Feature Architecture + Composable

Dado que vamos a hacer un proyecto sencillo, vamos a utilizar la **feature architecture** para organizar
los ficheros, y vamos a unirla al uso de **Composables** para hacer nuestros componentes ligeros y lógica reusable.

> **Feature architecture vs Layers** 👉 https://dev.to/smotastic/layer-vs-feature-architecture-3cko

## Creando el formulario básico

### VeeValidate

Vamos a utilizar el paquete **VeeValidate** para construir los forms.
Primero de todo, vamos a realizar la instalación

> ```bash
> npm i vee-validate
> ```
>


> 📋 Como estamos utilizando Vue3, vamos a instalar la v4 de VeeValidate, pero en caso de utilizar Vue2, deberiamos
> instalar
> la v3.
> Más información aquí 👉https://vee-validate.logaretm.com/v4/

### 2. Formulario con VeeValidate

Según la documentación de **VeeValidate** (a la que llamaremos **VV** a partir de ahora), existen dos maneras
de componer los formularios:

1. Mediante componentes
2. Mediante composition API

```text
vee-validate makes use of two flavors to add validation to your forms.

Composition API: This is the best way to use vee-validate as it allows seamless integrations with your existing UI, or any 3rd party component library.
Higher-order components (HOC): This approach is easy to use and is strictly used within the template, you can use it if you have simple forms and don’t want to write a lot of JavaScript.
```

En esta aplicación vamos a hacerlo mediante **composition**, únicamente por preferencia personal.

#### Componente del formulario

Si vamos a la 🌏 <a href="https://vee-validate.logaretm.com/v4/">página principal</a>, donde reza el título **Do more
with less**, y cambiamos la opción a
**Composition API**, veremos que aparece un formulario hecho con este aspecto:

````vue

<script setup>
  import {useForm} from 'vee-validate';
  import * as yup from 'yup';

  const schema = yup.object({
    email: yup.string().email().required(),
    password: yup.string().min(6).required(),
  });

  const {defineField, errors, handleSubmit} = useForm({
    validationSchema: schema,
  });

  const [email, emailAttrs] = defineField('email');
  const [password, passwordAttrs] = defineField('password');

  const onSubmit = handleSubmit(values => {
    alert(JSON.stringify(values, null, 2));
  });
</script>

<template>
  <form @submit="onSubmit">
    <input v-model="email" v-bind="emailAttrs" name="email" type="email"/>
    <span>{{ errors.email }}</span>

    <input v-model="password" v-bind="passwordAttrs" name="password" type="password"/>
    <span>{{ errors.password }}</span>

    <button>Submit</button>
  </form>
</template>
````

Para la validación de los campos se ha utilizado `yup` (que es otra librería como `zod`). Pero lo importante es ver *
*cómo se definen los campos que forman
parte del formulario**:

```javascript
    const [email, emailAttrs] = defineField('email');
const [password, passwordAttrs] = defineField('password');
```

Estos dos métodos, importados desde la librería de ``VeeValidate``, nos permiten definir esos campos. Serían el
equivalente al uso de ``register`` en ``react-hook-forms``.

El resultado de nuestro formulario será algo parecido a esto.

#### 2. Creación del componente

La primera problemática a la que nos enfrentamos es cómo llamar a los componentes. Sin alejarnos de la línea que sigue
React, los componentes de
Vue deben seguir las siguientes reglas para ajustarnos a las mejores prácticas posibles:

1. Los componentes deben utilizar la nomenclatura **PascalCase** o **kebab-case**.

> Puedes leer más sobre el tema aquí:
> 👉 https://learnvue.co/articles/vue-best-practices#_7-use-pascalcase-or-kebab-case-for-components

2. Si los componentes son únicos en toda la aplicación (es decir, que solo vayas a usar uno), deben ir precedidos por la
   palabra ``The``:

> Puedes leer más aquí:
> 👉 https://learnvue.co/articles/vue-best-practices#_9-components-declared-and-used-once-should-have-the-prefix-the


De momento, vamos a asumir que nuestro componente Form pueda repetirse más de una vez, y vamos a crearlo bajo el nombre
de ```FormBuilder```; no porque sea
un constructor génerico de forms, sino porque va a ser a "plantilla" que van a usar los consumidores de la aplicación
para crear **su** form deseado.

````
src/
└── features/
    └── Form
        └── FormBuilder.vue
````

Vamos a realizar esta estructura: crearemos el componente **dentro** de una carpeta llamada Form, pues ahí añadiremos
otra lógica necesaria o componentes relacionados.

#### Template básica del componente

Cuando creas un componente en ``vue``, se genera con esta template básica:

````vue

<script setup lang="ts">

</script>

<template>

</template>

<style scoped>

</style>

````

- El primer bloque, ``script``, contendrá la lógica del componente, ya sea desarrollada en el mismo o importada de otros
  ficheros
- El segundo bloque, ``template``, la parte visual del componente: las etiquetas u otros componentes.
- El tercer bloque, ``style``, los estilos.

#### Creación del componente

Vamos a crear de momento algo muy básico: Un simple formulario que te pida cuál es el nombre del campo y qué tipo de
datos quiere que recoja:

```vue

<script setup lang="ts">

</script>

<template>
  <form>
    <label>Introduce el nombre del campo
      <input type="text"/>
    </label>

    <label>Introduce qué tipo de datos admite el campo
      <select>
        <option>Texto</option>
        <option>Números</option>
        <option>Selección única</option>
        <option>Selección múltiple</option>
      </select>
    </label>
  </form>

</template>

<style scoped>

</style>
```

> 📋 Más adelante incluiremos la librería Vue I18n para las traducciones: 👉 https://vue-i18n.intlify.dev/


Limpiamos el componente por defecto `App.vue` para que quede solo con la etiqueta `<template>` (apertura y cierre) y
llamamos
al nuevo componente:

````vue

<script setup lang="ts">
  import FormBuilder from "./features/Form/FormBuilder.vue";
</script>

<template>
  <FormBuilder/>
</template>
````

#### Definición del formulario: Composition API

> 🌏https://vee-validate.logaretm.com/v4/guide/composition-api/getting-started/

Al igual que como ocurre en ``react``, la función `useForm` genera un ``context`` (contexto)
para el formulario en ciernes: eso quiere decir que **todos los controladores que estén bajo el formulario asociado
a ese contexto, pertenecerán al mismo**. Esto es muy útil cuando quieres **componetizar** para redistribuir la lógica de
gestión
del mismo y así evitar tener un componente **demasiado** **grande** y **díficil de gestionar**.

> Calling useForm creates a
> form context in the component and provides it
> for any child component that injects it.
> This means you should stick to calling useForm once in a component.

```typescript
import {useForm} from "vee-validate";

const {} = useForm();
```

##### 1. Cómo crear los controladores

> 🌏 https://vee-validate.logaretm.com/v4/guide/composition-api/custom-inputs/

```useForm``` nos permite acceder a una función llamada ``defineField`` que nos permitirá definir el nombre asociado
al controlador:

```typescript
import {useForm} from "vee-validate";

const {defineField} = useForm();
defineField()
```

El parámetro que admite esta función es un ``string`` que corresponderá al **nombre del controlador a asignar**.

Por ejemplo:

```vue

<script setup lang="ts">
  import {useForm} from "vee-validate";

  const {defineField} = useForm();
  defineField("fieldName")

</script>

<template>
  <form>
    <label>Introduce el nombre del campo
      <input type="text" v-model="fieldName"/>
    </label>

    <label>Introduce qué tipo de datos admite el campo
      <select>
        <option>Texto</option>
        <option>Números</option>
        <option>Selección única</option>
        <option>Selección múltiple</option>
      </select>
    </label>

    <button>Agregar campo</button>
  </form>
</template>

<style scoped>

</style>
```

##### ¿Qué son v-model y v-bind?

``v-model`` y ``v-bind`` son **dos directivas** que permiten interactuar con los elementos con datos dinámicos.

- ``v-model`` permite hacer un ``two-way-binding``: es decir, **que el dato asignado a un elemento se ve afectado por
  éste, y viceversa**.
- ``v-bind`` permite hacer ``bind`` de **uno o más atributos**.

En este contexto:

````vue

<script setup lang="ts">
  import {useForm} from "vee-validate";

  const {defineField} = useForm();
  const [fieldName, fieldNameAttrs] = defineField('fieldName')

</script>

<template>
  <form>
    <label>Introduce el nombre del campo
      <input type="text" v-model="fieldName" v-bind="fieldNameAttrs"/>
    </label>
  </form>
</template>
````

La función ``defineField`` nos devuelve **dos propiedades** al utilizarla:

- ``fieldName``, que será la variable donde **recogeremos** el dato introducido por el controlador.
- y ``fieldNameAttrs``, que contendrá las variables de validación **que especifiquemos al definir el controlador**.

Para añadir una regla de validación:

```typescript
const [fieldName, fieldNameAttrs] = defineField('fieldName', {
    props: function (state) {
        console.log("state", state)
        return {}
    }
})
```

Debemos extender la función ``defineProps`` para poder acceder a la propiedad ``props``, la cual nos permite recoger el
valor ``state``, donde se almacenan
**todos los datos vinculados al controlador**:

```typescript
{
    dirty:false // <--- Nos indica si el valor original del controlador ha sido modificado
    errors:[]
    initialValue:undefined
    label:undefined
    path:"fieldName" // <--- Aquí tenemos definido el nombre de nuestro controlador
    pending:false
    required:false
    touched:false
    valid:true
    validated:false
    value:undefined
}
```

Así que si hacemos algo así:

````typescript
const [fieldName, fieldNameAttrs] = defineField('fieldName', {
    props: _ => ({
        required: true,
    })
})
````

Al asignar las propiedades mediante ``fieldNameAttrs``, la propiedad ``required`` a la que hemos ahora _setteado_ con el
valor ``true``, será asignada
al controlador, dándonos como resultado que si intentamos _submitear_ el formulario, nos saltará el error de que el
campo es requerido.

Sin embargo, definir para **cada campo la misma regla** resulta un poco tedioso y engorroso, por lo que ``vee-validate``
nos ofrece otra manera de establecer unas reglas de manera
**global**:

> 🌏 https://vee-validate.logaretm.com/v4/guide/global-validators/

Según la documentación, mediante la función ``defineRule`` podemos definir una **regla** que aplique a los controladores
que deseemos:

```js
import {defineRule} from 'vee-validate';

defineRule('required', value => {
    if (!value || !value.length) {
        return 'This field is required';
    }
    return true;
});
```

De hecho la propia documentación nos facilita la regla para los campos requeridos.

Vamos a probarla tal cual.

Creemos un fichero ``validation.ts`` al nivel de nuestro form:

````
src/
└── features/
    └── Form
        └── FormBuilder.vue
        └── validations.ts
````

Y dentro copiemos la función que nos ha dado la documentación:

````typescript
import {defineRule} from 'vee-validate';

defineRule('required', (value: string) => {
    if (!value || !value.length) {
        return 'This field is required';
    }
    return true;
});
````

Al igual que cuando definimos el campo, tenemos que **definir** el nombre de la regla (a la que se ha denominado
``required``) y definir qué condiciones deben cumplirse
para que ésta sea válida.

Para poder utilizarla en el formulario, debemos extender la parte en la que definimos el mismo:

````typescript
const {defineField, handleSubmit} = useForm<{ fieldName: string }>({
    validationSchema: {
        fieldName: 'required'
    }
});
````

> 📝 VeeValidate tiene una buena **compenetración** con librerías como ``zod`` o ``yup`` que facilitan la validación de
> los formularios,
> pero para este proyecto no las utilizaremos porque la idea es aprender bien las bases de la creación de formularios.

Usamos `'required'` como `valor` porque es la **denominación** que le dimos a la regla.

Para obtener los campos que hallan fallado, la función `useForm` nos facilita de un objeto donde contiene los errores
para
**cada campo** llamado `errors`.

Si escribimos: `errors.fieldName` obtendremos el mensaje de error asociado.

> 🌏 https://vee-validate.logaretm.com/v4/guide/composition-api/handling-forms#errors

> 📝 También se nos facilita otro objeto llamado `errorBags` que contiene un array de **todos los errores** de validación
> asociados
> al campo que deseemos. Dependiendo de lo que necesitemos, precisaremos de usar de uno o de otro.

##### Recoger los datos del formulario: ```handleSubmit```

Los datos que se plasmen en el formulario pueden ser recogidos mediante la función `handleSubmit`. Esta función recibe
por parámetro a ``data`` del formulario,
y para acceder a ella podemos hacer algo como esto:

````typescript
const { handleSubmit } = useForm();
const onSubmit = handleSubmit((data) => {
    console.log("data", data)
})
````

Tenemos que almacenar la función en una constante para luego vincularla al formulario:

```vue
<form @submit="onSubmit">
  ...
</form>
```


> Para _bindear_ atributos a los elementos html usamos la directiva ``v-bind``, y para _triggerear_ acciones debemos hacerlo utilizando ``@``, al contrario
> que en react, que se usa el prefijo `on` por delante de la acción a disparar.

📝 Es posible que necesitemos desarrollar una lógica bastante densa dentro del ``submit``, y a mí, personalmente, tenerlo en el mismo componente no me gusta
porque si luego éste crece mucho, se hace difícil de leer. Podemos aislar la lógica de la siguiente manera:

````
src/
└── features/
    └── Form
        └── FormBuilder.vue
        └── validations.ts
        └── handler.ts
````

Creamos un fichero llamado ``handler.ts`` donde desarrollaremos esa lógica:

```typescript
import {FormBuilderType} from "./types.ts";

export const handleData = (data: FormBuilderType) => {
    console.log("data", data)
}
```

Y luego esa función la llamamos dentro del ``handleSubmit`` que nos facilita el componente:

```typescript
import {handleData} from "./handler.ts";
const { handleSubmit } = useForm();
const onSubmit = handleSubmit((data) => handleData(data))
```


> 📝 `useForm` admite tipos genéricos, así que podemos crear un ``type`` con el que definamos los campos de los que se
> compone el formulario:
> 
> ````
> src/
> └── features/
> └── Form
> └── FormBuilder.vue
> └── validations.ts
> └── handler.ts
> └── types.ts
> ````
> 
> ```typescript
> export type FormBuilderType = {
>    fieldName: string,
>    fieldType: string,
> }
> ```
> 
> 
> ```typescript
> const { defineField, handleSubmit, errors } = useForm<FormBuilderType>({
> validationSchema: {
> fieldName: 'required'
> }
> });
> ```


Así que ahora nuestro componente luce asi:

```vue
<script setup lang="ts">
import {useForm} from "vee-validate";
import {handleData} from "./handler.ts";
import {FormBuilderType} from "./types.ts";

const { defineField, handleSubmit, errors } = useForm<FormBuilderType>({
  validationSchema: {
    fieldName: 'required'
  }
});
const [fieldName, fieldNameAttrs] = defineField('fieldName')
const [fieldType, fieldTypeAttrs] = defineField('fieldType')
const onSubmit = handleSubmit((data) => handleData(data))

</script>
<template>
  <form @submit="onSubmit">
    <label>Introduce el nombre del campo
      <input type="text"  v-model="fieldName" v-bind="fieldNameAttrs" />
      <small> {{ errors.fieldName }}</small>
    </label>

    <label>Introduce qué tipo de datos admite el campo
      <select v-model="fieldType" v-bind="fieldTypeAttrs">
        <option>Texto</option>
        <option>Números</option>
        <option>Selección única</option>
        <option>Selección múltiple</option>
      </select>
    </label>

    <button>Agregar campo</button>
  </form>
</template>

<style scoped>

</style>
```

### 3. Vuetify, embellecer la aplicación.

> 🌏 https://vuetifyjs.com/en/getting-started/installation/#existing-projects

Vuetify nos permite empezar un proyecto o **instalarlo en uno ya creado** (que es nuestro caso.)

```bash
npm i vuetify
```

Ahora debemos modificar el fichero main para que se cree de entrada los componentes y directivas:

````typescript
import './features/Form/validations.ts'
import './style.css'
import App from './App.vue'
import { createApp } from 'vue'

// Vuetify
import 'vuetify/styles'
import { createVuetify } from 'vuetify'
import * as components from 'vuetify/components'
import * as directives from 'vuetify/directives'

// Components

const vuetify = createVuetify({
    components,
    directives,
})

createApp(App).use(vuetify).mount('#app')
````

> ‼️Ten cuidado porque la configuración recomendada tiene un **fallo**. Cuando importamos los estilos:
> ``import 'vuetify/styles'``
> Nos da el siguiente error:
> ```Vue: Cannot find module vuetify/styles or its corresponding type declarations.```
> Para arreglar esto, debenos **extender el import** hasta ``main.css``
> 
> ```` import 'vuetify/styles/main.css'````
> 
> 👉 Aquí tienes la solución al respecto: https://www.reddit.com/r/vuejs/comments/1i4zee2/cannot_find_module_vuetifystyles_or_its/

