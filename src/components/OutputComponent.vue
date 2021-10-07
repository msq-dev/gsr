<template>
  <div class="output-component">
    <span :class="labelClass">
      <span v-html="label"></span>
    </span>
    <span class="result">
      <span v-html="operator"></span> {{ germanize(value) }} {{ unit }}
    </span>
  </div>
</template>

<script>
export default {
  name: "OutputComponent",
  props: {
    value: [String, Number, Boolean],
    label: String,
    unit: String,
    operator: String,
    labelClass: String
  },
  methods: {
    germanize(input) {
      input = String(input).replace(",", ".")
      input = input || "0"
      return parseFloat(input).toFixed(2).replace(".", ",")
    }
  }
}
</script>

<style scoped>
.output-component {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(15ch, 2fr);
  margin-bottom: 1rem;
  width: 100%;
  align-items: bottom;  
}

.result {
  font-family: "Azeret Mono", monospace;
  text-align: right;
}

.last-calc,
.last-calc + span {
  padding-bottom: 0.25rem;
  border-bottom: 1px solid #dedede;
  margin-bottom: 0.25rem;
}

.final-result,
.final-result + span {
  font-size: 110%;
  font-weight: bold;
}
</style>