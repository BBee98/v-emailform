<script setup lang="ts">
import {useForm} from "vee-validate";
import {handleData} from "./handler.ts";
import {FormBuilderType} from "./types.ts";

const { defineField, handleSubmit } = useForm<FormBuilderType>({
  validationSchema: {
    fieldName: 'required'
  }
});
const [fieldName, fieldNameAttrs] = defineField('fieldName')
const onSubmit = handleSubmit((data, error) => handleData(data, error))

</script>
<template>
  <form @submit="onSubmit">
    <label>Introduce el nombre del campo
      <input type="text"  v-model="fieldName" v-bind="fieldNameAttrs" />
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