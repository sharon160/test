<template>
  <v-app>
    <v-main>
      <HelloWorld/>
      <!-- <runtime-report></runtime-report> -->
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';
// import RuntimeReport from './components/RuntimeReport.vue';

export default {
  name: 'App',
  components: {
    HelloWorld,
    // RuntimeReport
  },

  data() {
    return {

    }
  }
};
</script>
