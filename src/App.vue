<template>
  <v-app>
    <v-main>
      <Weather />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Weather from './components/Weather.vue'

export default defineComponent({
  name: 'App',
  components: {
    Weather,
  },

  data() {
    return {
      //
    }
  },
})
</script>
