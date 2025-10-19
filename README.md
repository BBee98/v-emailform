# Aprendiendo Vue

## Ordenando el proyecto: Arquitectura

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


> 📋 Como estamos utilizando Vue3, vamos a instalar la v4 de VeeValidate, pero en caso de utilizar Vue2, deberiamos instalar
> la v3.
> Más información aquí 👉https://vee-validate.logaretm.com/v4/

### 1. Prólogo

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

Si vamos a la 🌏 <a href="https://vee-validate.logaretm.com/v4/">página principal</a>, donde reza el título **Do more with less**, y cambiamos la opción a
 **Composition API**, veremos que aparece un formulario hecho con este aspecto:

````vue
<script setup>
    import { useForm } from 'vee-validate';
    import * as yup from 'yup';

    const schema = yup.object({
    email: yup.string().email().required(),
    password: yup.string().min(6).required(),
});

    const { defineField, errors, handleSubmit } = useForm({
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
    <input v-model="email" v-bind="emailAttrs" name="email" type="email" />
    <span>{{ errors.email }}</span>

    <input v-model="password" v-bind="passwordAttrs" name="password" type="password" />
    <span>{{ errors.password }}</span>

    <button>Submit</button>
</form>
</template>
````

Para la validación de los campos se ha utilizado `yup` (que es otra librería como `zod`). Pero lo importante es ver **cómo se definen los campos que forman
parte del formulario**:

```javascript
    const [email, emailAttrs] = defineField('email');
    const [password, passwordAttrs] = defineField('password');
```

Estos dos métodos, importados desde la librería de ``VeeValidate``, nos permiten definir esos campos. Serían el equivalente al uso de ``register`` en ``react-hook-forms``.

El resultado de nuestro formulario será algo parecido a esto.

#### 2. Creación del componente

La primera problemática a la que nos enfrentamos es cómo llamar a los componentes. Sin alejarnos de la línea que sigue React, los componentes de 
Vue deben seguir las siguientes reglas para ajustarnos a las mejores prácticas posibles:

1. Los componentes deben utilizar la nomenclatura **PascalCase** o **kebab-case**.

> Puedes leer más sobre el tema aquí: 👉 https://learnvue.co/articles/vue-best-practices#_7-use-pascalcase-or-kebab-case-for-components

2. Si los componentes son únicos en toda la aplicación (es decir, que solo vayas a usar uno), deben ir precedidos por la palabra ``The``:

> Puedes leer más aquí: 👉 https://learnvue.co/articles/vue-best-practices#_9-components-declared-and-used-once-should-have-the-prefix-the


De momento, vamos a asumir que nuestro componente Form pueda repetirse más de una vez, y vamos a crearlo bajo el nombre de ```FormBuilder```; no porque sea
un constructor génerico de forms, sino porque va a ser a "plantilla" que van a usar los consumidores de la aplicación para crear **su** form deseado.

````
src/
└── features/
    └── Form
        └── FormBuilder.vue
````

Vamos a realizar esta estructura: crearemos el componente **dentro** de una carpeta llamada Form, pues ahí añadiremos otra lógica necesaria o componentes relacionados.

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

- El primer bloque, ``script``, contendrá la lógica del componente, ya sea desarrollada en el mismo o importada de otros ficheros
- El segundo bloque, ``template``, la parte visual del componente: las etiquetas u otros componentes.
- El tercer bloque, ``style``, los estilos.


#### Creación del componente

Vamos a crear de momento algo muy básico: Un simple formulario que te pida cuál es el nombre del campo y qué tipo de datos quiere que recoja:

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


Limpiamos el componente por defecto `App.vue` para que quede solo con la etiqueta `<template>` (apertura y cierre) y llamamos
al nuevo componente:

````vue
<script setup lang="ts">
import FormBuilder from "./features/Form/FormBuilder.vue";
</script>

<template>
  <FormBuilder />
</template>
````
