<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-paint-palette.svg" alt="Theme" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('admin:theme.title')}}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">{{$t('admin:theme.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInRight" color="success" depressed @click="save" large :loading="loading">
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
        <v-form class="pt-3">
          <v-layout row wrap>
            <v-flex lg6 xs12>
              <v-card class="animated fadeInUp">
                <v-toolbar color="primary" dark dense flat>
                  <v-toolbar-title class="subtitle-1">{{$t('admin:theme.title')}}</v-toolbar-title>
                </v-toolbar>
                <v-card-text>
                  <v-select :items="themes" outlined prepend-icon="mdi-palette" v-model="config.theme" :label="$t(`admin:theme.siteTheme`)" persistent-hint :hint="$t(`admin:theme.siteThemeHint`)">
                    <template slot="item" slot-scope="data">
                      <v-list-item-avatar>
                        <v-icon class="blue--text" dark>mdi-image-filter-frames</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title v-html="data.item.text"></v-list-item-title>
                        <v-list-item-sub-title v-html="data.item.author"></v-list-item-sub-title>
                      </v-list-item-content>
                    </template>
                  </v-select>
                  <v-select class="mt-3" :items="iconsets" outlined prepend-icon="mdi-paw" v-model="config.iconset" :label="$t(`admin:theme.iconset`)" persistent-hint :hint="$t(`admin:theme.iconsetHint`)"></v-select>
                  <v-divider class="mt-3"></v-divider>
                  <v-switch inset v-model="darkMode" :label="$t(`admin:theme.darkMode`)" color="primary" persistent-hint :hint="$t(`admin:theme.darkModeHint`)"></v-switch>
                </v-card-text>
              </v-card>
              <v-card class="mt-3 animated fadeInUp wait-p1s">
                <v-toolbar color="primary" dark dense flat>
                  <v-toolbar-title class="subtitle-1">{{$t(`admin:theme.options`)}}</v-toolbar-title>
                </v-toolbar>
                <v-card-text>
                  <v-select :items="tocPositions" outlined prepend-icon="mdi-border-vertical" v-model="config.tocPosition" label="Table of Contents Position" persistent-hint hint="Select whether the table of contents is shown on the left, right or not at all."></v-select>
                </v-card-text>
              </v-card>
            </v-flex>
            <v-flex lg6 xs12>
              <v-card class="animated fadeInUp wait-p2s">
                <v-toolbar color="primary" dark dense flat>
                  <v-toolbar-title class="subtitle-1">{{$t(`admin:theme.codeInjection`)}}</v-toolbar-title>
                </v-toolbar>
                <v-card-text>
                  <v-textarea class="is-monospaced" v-model="config.injectCSS" :label="$t(`admin:theme.cssOverride`)" outlined color="primary" persistent-hint :hint="$t(`admin:theme.cssOverrideHint`)" auto-grow></v-textarea>
                  <i18next class="caption pl-2 ml-1" path="admin:theme.cssOverrideWarning" tag="div"><strong class="red--text" place="caution">{{$t('admin:theme.cssOverrideWarningCaution')}}</strong><code place="cssClass">.contents</code></i18next>
                  <v-textarea class="is-monospaced mt-3" v-model="config.injectHead" :label="$t(`admin:theme.headHtmlInjection`)" outlined color="primary" persistent-hint :hint="$t(`admin:theme.headHtmlInjectionHint`)" auto-grow></v-textarea>
                  <v-textarea class="is-monospaced mt-2" v-model="config.injectBody" :label="$t(`admin:theme.bodyHtmlInjection`)" outlined color="primary" persistent-hint :hint="$t(`admin:theme.bodyHtmlInjectionHint`)" auto-grow></v-textarea>
                </v-card-text>
              </v-card>
            </v-flex>
          </v-layout>
        </v-form>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'
import { sync } from 'vuex-pathify'

import themeConfigQuery from 'gql/admin/theme/theme-query-config.gql'
import themeSaveMutation from 'gql/admin/theme/theme-mutation-save.gql'

export default {
  data() {
    return {
      loading: false,
      themes: [
        { text: 'Default', author: 'requarks.io', value: 'default', isInstalled: true, installDate: '', updatedAt: '' }
      ],
      iconsets: [
        { text: 'Material Design Icons (default)', value: 'mdi' },
        { text: 'Font Awesome 5', value: 'fa' },
        { text: 'Font Awesome 4', value: 'fa4' }
      ],
      config: {
        theme: 'default',
        darkMode: false,
        iconset: '',
        tocPosition: 'left',
        injectCSS: '',
        injectHead: '',
        injectBody: ''
      },
      darkModeInitial: false
    }
  },
  computed: {
    darkMode: sync('site/dark'),
    headers() {
      return [
        {
          text: this.$t('admin:theme.downloadName'),
          align: 'left',
          value: 'text'
        },
        {
          text: this.$t('admin:theme.downloadAuthor'),
          align: 'left',
          value: 'author'
        },
        {
          text: this.$t('admin:theme.downloadDownload'),
          align: 'center',
          value: 'value',
          sortable: false,
          width: 100
        }
      ]
    },
    tocPositions () {
      return [
        { text: 'Left (default)', value: 'left' },
        { text: 'Right', value: 'right' },
        { text: 'Hidden', value: 'off' }
      ]
    }
  },
  watch: {
    'darkMode' (newValue, oldValue) {
      this.$vuetify.theme.dark = newValue
    }
  },
  mounted() {
    this.darkModeInitial = this.darkMode
  },
  beforeDestroy() {
    this.darkMode = this.darkModeInitial
    this.$vuetify.theme.dark = this.darkModeInitial
  },
  methods: {
    async save () {
      this.loading = true
      this.$store.commit(`loadingStart`, 'admin-theme-save')
      try {
        const respRaw = await this.$apollo.mutate({
          mutation: themeSaveMutation,
          variables: {
            theme: this.config.theme,
            iconset: this.config.iconset,
            darkMode: this.darkMode,
            tocPosition: this.config.tocPosition,
            injectCSS: this.config.injectCSS,
            injectHead: this.config.injectHead,
            injectBody: this.config.injectBody
          }
        })
        const resp = _.get(respRaw, 'data.theming.setConfig.responseResult', {})
        if (resp.succeeded) {
          this.darkModeInitial = this.darkMode
          this.$store.commit('showNotification', {
            message: 'Theme settings updated successfully.',
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-theme-save')
      this.loading = false
    }
  },
  apollo: {
    config: {
      query: themeConfigQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.theming.config,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-theme-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>
.v-textarea.is-monospaced textarea {
  font-family: 'Roboto Mono', 'Courier New', Courier, monospace;
  font-size: 13px;
  font-weight: 600;
  line-height: 1.4;
}
</style>
