<template>  
  <v-app>
    <div class="onboarding">
      <div class="onboarding-content"><img class="animated fadeIn" src="/_assets/svg/logo-wikijs.svg" alt="Wiki.js">
        <div class="headline animated fadeInUp">{{ $t('welcome.title') }}</div>
        <div class="subtitle-1 mt-3 animated fadeInUp wait-p1s">{{ $t('welcome.subtitle') }}</div>
        <div>
          <v-btn class="mt-5 mx-3 animated fadeInUp wait-p2s" color="primary" :href="`/e/` + locale + `/home`" x-large>
            <v-icon left>mdi-plus</v-icon><span>{{ $t('welcome.createhome') }}</span>
          </v-btn>
          <v-btn class="mt-5 mx-3 animated fadeInUp wait-p3s" color="primary" href="/a" x-large>
            <v-icon left>mdi-view-dashboard</v-icon><span>{{ $t('welcome.goadmin') }}</span>
          </v-btn>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>

export default {
  props: {
    locale: {
      type: String,
      default: 'en'
    }
  },
  data() {
    return { }
  }
}
</script>

<style lang='scss'>

</style>
