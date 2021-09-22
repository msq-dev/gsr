<template>
  <div class="input-component">
    <label :for="label"><span v-html="label"></span></label>
    <input
      ref="input"
      type="text"
      :id="label"
      inputmode="decimal"
      :value="value"
      @input="$emit('input', $event.target.value)"
      @keyup.enter="$emit('validate')"
    >
    <span>{{ unit }}</span>
  </div>
</template>

<script>
export default {
  name: "InputComponent",
  props: {
    value: String,
    label: String,
    unit: String
  }
}
</script>

<style scoped>
.input-component {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(17ch, 2fr) 1ch;
  align-items: center;
  margin-bottom: 1rem;
  width: 100%;
}

label {
  margin-right: 0.5rem;
}

input {
  font-family: inherit;
  font-size: inherit;
  background: rgba(10, 36, 51, 0.1);
  border: none;
  border-radius: 10px;
  margin-right: 0.5rem;
  padding-right: 0.5rem;
  text-align: right;
  height: 2.2rem;
}

input:focus {
  outline: none;
  box-shadow: 0 0 0 1px var(--clr-accent);
}

input.invalid {
  box-shadow: 0 0 0 2px tomato !important;
}

</style>