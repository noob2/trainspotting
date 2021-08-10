<template>
  <div id="app">
    <Trainspotting />
  </div>
</template>

<script>
import Trainspotting from './components/Trainspotting.vue'

export default {
  name: 'App',
  components: {
    Trainspotting: () => import('./components/Trainspotting.vue')
  }
}
</script>
