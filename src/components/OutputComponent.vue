<template>
  <div class="output-component">
    <span :class="labelClass">
      <span v-html="label"></span>
    </span>
    <span class="result">
      <span v-html="operator"></span> {{ germanize(clean ? value : cleanString(value)) }} {{ unit }}
    </span>
  </div>
</template>

<script>
export default {
  name: "OutputComponent",
  props: {
    value: String,
    label: String,
    unit: String,
    operator: String,
    clean: Boolean,
    labelClass: String
  },
  methods: {
    cleanString(input) {
      if (input === "") { input = "0" }
      input = input.replaceAll(" ", "")
      input = input.replaceAll(",", ".")
      return input
    },

    germanize(number) {
      return parseFloat(number).toFixed(2).replace(".", ",")
    }
  }
}
</script>

<style scoped>
.output-component {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(20ch, 2fr);
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