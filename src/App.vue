<template>
  <v-app>
    <v-main>
      <StatusReport/>
      <!-- <runtime-report></runtime-report> -->
      <!-- <faults></faults> -->
    </v-main>
  </v-app>
</template>

<script>
import StatusReport from './components/StatusReport';
// import RuntimeReport from './components/RuntimeReport.vue';
// import Faults from './components/Faults.vue';

export default {
  name: 'App',
  components: {
    StatusReport,
    // RuntimeReport,
    // Faults
  },

  data() {
    return {

    }
  }
};
</script>
