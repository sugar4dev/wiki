<template>  
  <v-app>
    <div class="notfound">
      <div class="notfound-content"><img class="animated fadeIn" src="/_assets/svg/icon-delete-file.svg" alt="Not Found">
        <div class="headline">{{$t('notfound.title')}}</div>
        <div class="subheading mt-3">{{$t('notfound.subtitle')}}</div>
        <v-btn class="mt-5" color="red lighten-4" href="/" large outlined>
          <v-icon left>mdi-home</v-icon><span>{{$t('notfound.gohome')}}</span>
        </v-btn>
      </div>
    </div>
  </v-app>
</template>

<script>

export default {
  data() {
    return { }
  }
}
</script>

<style lang='scss'>

</style>
