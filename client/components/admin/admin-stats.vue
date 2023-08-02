<template>  
  <v-container fluid fill-height>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header-icon">
          <v-icon size="80" color="grey lighten-2">show_chart</v-icon>
        </div>
        <div class="headline primary--text">Statistics</div>
        <div class="subtitle-1 grey--text">Useful information about your wiki</div>
        <div class="pa-3">
          <fingerprint-spinner :animation-duration="1500" :size="128" color="#e91e63"></fingerprint-spinner>
          <div class="caption pink--text mt-3">Compiling latest data...</div>
        </div>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import { FingerprintSpinner } from 'epic-spinners'

export default {
  components: {
    FingerprintSpinner
  },
  data() {
    return {}
  }
}
</script>

<style lang='scss'>

</style>
