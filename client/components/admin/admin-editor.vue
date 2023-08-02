<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img src="/_assets/svg/icon-web-design.svg" alt="Editor" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text">Editor</div>
            <div class="subtitle-1 grey--text">Configure the content editors 
              <v-chip class="white--text" label color="primary" small>coming soon</v-chip>
            </div>
          </div>
          <v-spacer></v-spacer>
          <v-btn outline color="grey" @click="refresh" large>
            <v-icon>refresh</v-icon>
          </v-btn>
          <v-btn color="success" @click="save" depressed large>
            <v-icon left>check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
        <v-card class="mt-3">
          <v-tabs color="grey darken-2" fixed-tabs slider-color="white" show-arrows dark>
            <v-tab key="settings">
              <v-icon>settings</v-icon>
            </v-tab>
            <v-tab key="code">Markdown</v-tab>
            <v-tab-item key="settings" :transition="false" :reverse-transition="false">
              <v-card class="pa-3" flat tile>
                <div class="body-2 grey--text text--darken-1">Select which editors to enable:</div>
                <div class="caption grey--text pb-2">Some editors require additional configuration in their dedicated tab (when selected).</div>
                <v-form>
                  <v-checkbox class="my-0" v-for="editor in editors" v-model="editor.isEnabled" :key="editor.key" :label="editor.title" color="primary" disabled hide-details></v-checkbox>
                </v-form>
              </v-card>
            </v-tab-item>
            <v-tab-item key="code" :transition="false" :reverse-transition="false">
              <v-card class="wiki-form pa-3" flat tile>
                <v-form>
                  <v-subheader>Editor Configuration</v-subheader>
                  <div class="body-1 ml-3">This editor has no configuration options you can modify.</div>
                </v-form>
              </v-card>
            </v-tab-item>
          </v-tabs>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      editors: [
        { title: 'API Docs', key: 'api', isEnabled: false },
        { title: 'Code', key: 'code', isEnabled: true },
        { title: 'Markdown', key: 'markdown', isEnabled: true },
        { title: 'Tabular', key: 'tabular', isEnabled: false },
        { title: 'Visual Builder', key: 'visual', isEnabled: false },
        { title: 'WikiText', key: 'wikitext', isEnabled: false }
      ]
    }
  },
  methods: {
    save() {},
    refresh() {}
  }
}
</script>

<style lang='scss'>

</style>
