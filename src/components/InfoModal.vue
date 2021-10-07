<template>
  <div class="overlay" @click.self="$emit('close-modal')">
    <div class="info">
      <span v-html="info.default"></span>
      <button class="btn" @click="$emit('close-modal')">Schließen</button>
    </div>    
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
.overlay {
  position: fixed;
  inset: 0;
  z-index: 5;
  display: flex;
  justify-content: center;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.8);
  overflow-y: scroll;
}

.info {
  position: absolute;
  top: 15%;
  z-index: 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  height: fit-content;
  background: #eee;
  border-radius: 10px;
  box-shadow: 0px 0px 19px 0px rgba(0,0,0,0.35);
  max-width: 70ch;
  padding: 1rem 2rem;
  margin: 1rem;
}

.info .btn {
  width: fit-content;
  margin: 1rem 0;
}

</style>