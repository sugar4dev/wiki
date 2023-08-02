<template>  
  <v-app>
    <div class="newpage">
      <div class="newpage-content"><img class="animated fadeIn" src="/_assets/svg/icon-delete-file.svg" alt="Not Found">
        <div class="headline">{{ $t('newpage.title') }}</div>
        <div class="subtitle-1 mt-3">{{ $t('newpage.subtitle') }}</div>
        <v-btn class="mt-5" :href="`/e/` + locale + `/` + path" x-large>
          <v-icon left>mdi-plus</v-icon><span>{{ $t('newpage.create') }}</span>
        </v-btn>
        <v-btn class="mt-5" color="purple lighten-3" href="javascript:window.history.go(-1);" outlined>
          <v-icon left>mdi-arrow-left</v-icon><span>{{ $t('newpage.goback') }}</span>
        </v-btn>
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
    },
    path: {
      type: String,
      default: 'home'
    }
  },
  data() {
    return { }
  }
}
</script>

<style lang='scss'>

</style>
