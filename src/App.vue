<template>
  <div style="margin: auto;">
    <h1>Random Room Generator</h1>
    <RoomCanvas />
  </div>
</template>

<script>
import RoomCanvas from "./components/RoomCanvas.vue";

export default {
  components: {
    RoomCanvas,
  },
};
</script>

<style>
h1 {
  text-align: center;
  margin-top: 20px;
}
#app{
  display: flex;
  justify-content: center;
}
</style>

