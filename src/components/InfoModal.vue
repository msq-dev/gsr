<template>
  <div class="info">
    <span v-html="info.default"></span>
    <button @click="$emit('close-modal')">Schließen</button>
  </div>
</template>

<script>
export default {
  name:"InfoModal",
  props: {
    infoFile: String
  },
  data() {
    return {
      info: require(`@/assets/info/${this.infoFile}`)
    }
  }
}
</script>

<style scoped>
.info {
  position: absolute;
  inset: 0;
  z-index: 10;
  background: #eee;
  border-radius: 10px;
  box-shadow: 0px 0px 19px 0px rgba(0,0,0,0.35);
  max-width: 66ch;
  height: fit-content;
  padding: 1rem 2rem;
  margin: 1rem;
}

.info button {
  padding: 1rem 2rem;
  margin: 1rem 0;
  font-family: inherit;
  font-size: inherit;
  color: #fff;
  background: var(--clr-accent);
  border: none;
  border-radius: 9999px;
  cursor: pointer;
}

</style>