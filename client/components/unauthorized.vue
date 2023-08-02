<template>  
  <v-app>
    <div class="unauthorized">
      <div class="unauthorized-content"><img class="animated fadeIn" src="/_assets/svg/icon-delete-shield.svg" alt="Unauthorized">
        <div class="headline">{{$t('unauthorized.title')}}</div>
        <div class="subtitle-1 mt-3">{{$t('unauthorized.action.' + action)}}</div>
        <v-btn class="mt-5" href="/login" x-large>
          <v-icon left>mdi-login</v-icon><span>{{$t('unauthorized.login')}}</span>
        </v-btn>
        <v-btn class="mt-5" color="red lighten-4" href="javascript:window.history.go(-1);" outlined>
          <v-icon left>mdi-arrow-left</v-icon><span>{{$t('unauthorized.goback')}}</span>
        </v-btn>
      </div>
    </div>
  </v-app>
</template>

<script>

export default {
  props: {
    action: {
      type: String,
      default: 'view'
    }
  },
  data() {
    return { }
  }
}
</script>

<style lang='scss'>

</style>
