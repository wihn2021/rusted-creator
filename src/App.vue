<template>
  <UnitEditor />
</template>

<script>
import UnitEditor from "./components/UnitEditor.vue";

export default {
  name: 'App',
  components: {
    UnitEditor
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
button {
  border-color: #BCC2D7;
  color: #EFF1FE;
  background-color: #194568;
}
button:hover {
  border-color: #EFF1FE;
  color: #EFF1FE;
  background-color: #546C8C;
}
</style>
