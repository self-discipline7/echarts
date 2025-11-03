<!-- App.vue -->
<template>
  <div id="app" style="background-color: #2A2A2A;">
    <ChinaMap :width="width" :height="height" />
  </div>
</template>

<script>
import ChinaMap from './components/ChinaMap.vue'

export default {
  name: 'App',
  components: {
    ChinaMap
  },
  data() {
    return {
      width: '800px',
      height: '600px'
    }
  }
}
</script>