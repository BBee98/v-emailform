<script setup lang="ts">
import {useForm} from "vee-validate";

const { defineField } = useForm<{fieldName: string}>();
const [fieldName, fieldNameAttrs] = defineField('fieldName', {
  props: _ => ({
    required: true,
  })
})

</script>

<template>
  <form>
    <label>Introduce el nombre del campo
      <input type="text"  v-model="fieldName" v-bind="fieldNameAttrs"/>
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