<template>
  <WebGL2D />
  <WebGL3D />
</template>

<script>
import WebGL2D from './components/2d.vue'
import WebGL3D from './components/3d.vue'
export default {
  components: {
    WebGL2D,
    WebGL3D
  }
}
</script>

<style scoped>

</style>
