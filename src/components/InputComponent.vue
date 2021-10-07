<template>
  <div class="input-component">

    <label class="input-label" :for="label">
      <span v-html="label"></span>
      <span v-if="hasInfo"
        class="info-icon"
        @click="showInfo = true"
      >
        <span v-html="infoIcon"></span>
        <!-- <font-awesome-icon icon="info" /> -->
      </span>
    </label>

    <select v-if="selectableInput"
      class="input-field"
      ref="input"
      :id="label"
      :value="value"
      @input="$emit('input', $event.target.value)"
      @change="$emit('validate')"
    >
      <option v-for="opt in selectOptions"
        :key="opt.rechtsform"
        :value="opt.freibetrag"
      >
        {{ opt.rechtsform }}
      </option>
    </select>

    <input v-else
      class="input-field"
      ref="input"
      type="text"
      :id="label"
      inputmode="decimal"
      :value="value"
      @input="$emit('input', $event.target.value)"
      @keyup.enter="$emit('validate')"
    >
    <span>{{ unit }}</span>


    <transition name="fade">
      <InfoModal
        v-if="showInfo"
        :infoFile="info"
        @close-modal="closeModal"
        v-on="$listeners"
      />
    </transition>

  </div>
</template>

<script>
import InfoModal from "./InfoModal.vue"

export default {
  name: "InputComponent",
  components: {
    InfoModal
  },
  props: {
    value: [String, Number],
    label: String,
    unit: String,
    hasInfo: Boolean,
    info: String,
    selectableInput: Boolean,
    selectOptions: Array
  },
  data() {
    return {
      showInfo: false,
      infoIcon : `<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="24" height="24" viewBox="0 0 24 24"><path d="M13,9H11V7H13M13,17H11V11H13M12,2A10,10 0 0,0 2,12A10,10 0 0,0 12,22A10,10 0 0,0 22,12A10,10 0 0,0 12,2Z" /></svg>`,
    }
  },
  methods: {
    closeModal() {
      this.showInfo = false
    }
  }
}
</script>

<style>
.input-component {
  display: grid;
  grid-template-columns: minmax(auto, 3fr) minmax(15ch, 2fr) 1ch;
  align-items: center;
  margin-bottom: 1rem;
  width: 100%;
}

.input-field {
  font-family: inherit;
  font-size: inherit;
  background: rgba(10, 36, 51, 0.1);
  border: none;
  border-radius: 0.6rem;
  margin-right: 0.5rem;
  padding-right: 0.5rem;
  text-align: right;
  height: 2.2rem;
}

.input-field:focus {
  outline: none;
  box-shadow: 0 0 0 1px var(--clr-accent);
}

.input-field.invalid {
  outline: none;
  box-shadow: 0 0 0 1px var(--clr-warning);
  /* border: 1px solid var(--clr-warning); */
}

.input-component select {
  appearance: none;
}



.info-icon svg {
  /* display: inline-block; */
  /* color: var(--clr-accent); */
  margin-left: 0.3rem;
  /* font-size: 60%; */
  width: 1rem;
  /* transform: translateY(-70%); */

  fill: var(--clr-accent);
  cursor: pointer;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity .2s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}

</style>