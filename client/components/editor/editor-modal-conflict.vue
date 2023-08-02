<template>  
  <v-card class="editor-modal-conflict animated fadeIn" flat tile>
    <div class="pa-4">
      <v-toolbar class="radius-7" flat color="indigo" style="border-bottom-left-radius: 0; border-bottom-right-radius: 0;" dark>
        <v-icon class="mr-3">mdi-merge</v-icon>
        <div class="subtitle-1">{{$t('editor:conflict.title')}}</div>
        <v-spacer></v-spacer>
        <v-btn outlined color="white" @click="useLocal" :title="$t(`editor:conflict.useLocalHint`)">
          <v-icon left>mdi-alpha-l-box</v-icon><span>{{$t('editor:conflict.useLocal')}}</span>
        </v-btn>
        <v-dialog v-model="isRemoteConfirmDiagShown" width="500">
          <template v-slot:activator="{ on }">
            <v-btn class="ml-3" outlined color="white" v-on="on" :title="$t(`editor:conflict.useRemoteHint`)">
              <v-icon left>mdi-alpha-r-box</v-icon><span>{{$t('editor:conflict.useRemote')}}</span>
            </v-btn>
          </template>
          <v-card>
            <div class="dialog-header is-short is-indigo">
              <v-icon class="mr-3" color="white">mdi-alpha-r-box</v-icon><span>{{$t('editor:conflict.overwrite.title')}}</span>
            </div>
            <v-card-text class="pa-4">
              <i18next class="body-2" tag="div" path="editor:conflict.overwrite.description"><strong place="refEditsLost">{{$t('editor:conflict.overwrite.editsLost')}}</strong></i18next>
            </v-card-text>
            <v-card-chin>
              <v-spacer></v-spacer>
              <v-btn outlined color="indigo" @click="isRemoteConfirmDiagShown = false">
                <v-icon left>mdi-close</v-icon><span>{{$t('common:actions.cancel')}}</span>
              </v-btn>
              <v-btn @click="useRemote" color="indigo" dark>
                <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.confirm')}}</span>
              </v-btn>
            </v-card-chin>
          </v-card>
        </v-dialog>
        <v-divider class="mx-3" vertical></v-divider>
        <v-btn outlined color="indigo lighten-4" @click="close">
          <v-icon left>mdi-close</v-icon><span>{{$t('common:actions.cancel')}}</span>
        </v-btn>
      </v-toolbar>
      <v-row class="indigo darken-1 body-2" no-gutters>
        <v-col class="pa-4">
          <v-icon class="mr-3" color="white">mdi-alpha-l-box</v-icon>
          <i18next class="white--text" tag="span" path="editor:conflict.localVersion"><em class="indigo--text text--lighten-4" place="refEditable">{{$t('editor:conflict.editable')}}</em></i18next>
        </v-col>
        <v-divider vertical></v-divider>
        <v-col class="pa-4">
          <v-icon class="mr-3" color="white">mdi-alpha-r-box</v-icon>
          <i18next class="white--text" tag="span" path="editor:conflict.remoteVersion"><em class="indigo--text text--lighten-4" place="refReadOnly">{{$t('editor:conflict.readonly')}}</em></i18next>
        </v-col>
      </v-row>
      <v-row class="grey lighten-2 body-2" no-gutters>
        <v-col class="px-4 py-2">
          <i18next class="grey--text text--darken-2" tag="em" path="editor:conflict.leftPanelInfo"><span place="date" :title="$options.filters.moment(checkoutDateActive, `LLL`)">{{ checkoutDateActive | moment('from') }}</span></i18next>
        </v-col>
        <v-divider vertical></v-divider>
        <v-col class="px-4 py-2">
          <i18next class="grey--text text--darken-2" tag="em" path="editor:conflict.rightPanelInfo"><strong place="authorName">{{latest.authorName}}</strong><span place="date" :title="$options.filters.moment(latest.updatedAt, `LLL`)">{{ latest.updatedAt | moment('from') }}</span></i18next>
        </v-col>
      </v-row>
      <v-row class="grey lighten-3 grey--text text--darken-3" no-gutters>
        <v-col class="pa-4">
          <div class="body-2"><strong class="indigo--text">{{$t('editor:conflict.pageTitle')}}</strong><strong class="pl-2">{{title}}</strong></div>
          <div class="caption"><strong class="indigo--text">{{$t('editor:conflict.pageDescription')}}</strong><span class="pl-2">{{description}}</span></div>
        </v-col>
        <v-divider vertical light></v-divider>
        <v-col class="pa-4">
          <div class="body-2"><strong class="indigo--text">{{$t('editor:conflict.pageTitle')}}</strong><strong class="pl-2">{{latest.title}}</strong></div>
          <div class="caption"><strong class="indigo--text">{{$t('editor:conflict.pageDescription')}}</strong><span class="pl-2">{{latest.description}}</span></div>
        </v-col>
      </v-row>
      <v-card class="radius-7" :light="!$vuetify.theme.dark" :dark="$vuetify.theme.dark">
        <div ref="cm"></div>
      </v-card>
    </div>
  </v-card>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'
import { sync, get } from 'vuex-pathify'

/* global siteConfig */

// ========================================
// IMPORTS
// ========================================

import '../../libs/codemirror-merge/diff-match-patch.js'

// Code Mirror
import CodeMirror from 'codemirror'
import 'codemirror/lib/codemirror.css'

// Language
import 'codemirror/mode/markdown/markdown.js'
import 'codemirror/mode/htmlmixed/htmlmixed.js'

// Addons
import 'codemirror/addon/selection/active-line.js'
import 'codemirror/addon/merge/merge.js'
import 'codemirror/addon/merge/merge.css'

export default {
  data() {
    return {
      cm: null,
      latest: {
        title: '',
        description: '',
        updatedAt: '',
        authorName: ''
      },
      isRemoteConfirmDiagShown: false
    }
  },
  computed: {
    editorKey: get('editor/editorKey'),
    activeModal: sync('editor/activeModal'),
    pageId: get('page/id'),
    title: get('page/title'),
    description: get('page/description'),
    updatedAt: get('page/updatedAt'),
    checkoutDateActive: sync('editor/checkoutDateActive')
  },
  methods: {
    close () {
      this.isRemoteConfirmDiagShown = false
      this.activeModal = ''
    },
    overwriteAndClose() {
      this.checkoutDateActive = this.latest.updatedAt
      this.$root.$emit('overwriteEditorContent')
      this.$root.$emit('resetEditorConflict')
      this.close()
    },
    useLocal () {
      this.$store.set('editor/content', this.cm.edit.getValue())
      this.overwriteAndClose()
    },
    useRemote () {
      this.$store.set('editor/content', this.latest.content)
      this.overwriteAndClose()
    }
  },
  async mounted () {
    let textMode = 'text/html'

    switch (this.editorKey) {
      case 'markdown':
        textMode = 'text/markdown'
        break
    }

    let resp = await this.$apollo.query({
      query: gql`
        query ($id: Int!) {
          pages {
            conflictLatest(id: $id) {
              id
              authorId
              authorName
              content
              createdAt
              description
              isPublished
              locale
              path
              tags
              title
              updatedAt
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      variables: {
        id: this.$store.get('page/id')
      }
    })
    resp = _.get(resp, 'data.pages.conflictLatest', false)

    if (!resp) {
      return this.$store.commit('showNotification', {
        message: 'Failed to fetch latest version.',
        style: 'warning',
        icon: 'warning'
      })
    }
    this.latest = resp

    this.cm = CodeMirror.MergeView(this.$refs.cm, {
      value: this.$store.get('editor/content'),
      orig: resp.content,
      tabSize: 2,
      mode: textMode,
      lineNumbers: true,
      lineWrapping: true,
      connect: null,
      highlightDifferences: true,
      styleActiveLine: true,
      collapseIdentical: true,
      direction: siteConfig.rtl ? 'rtl' : 'ltr'
    })
    this.cm.rightOriginal().setSize(null, 'calc(100vh - 265px)')
    this.cm.editor().setSize(null, 'calc(100vh - 265px)')
    this.cm.wrap.style.height = 'calc(100vh - 265px)'
  }
}
</script>

<style lang='scss'>
.editor-modal-conflict {
  position: fixed !important;
  top: 0;
  left: 0;
  z-index: 10;
  width: 100%;
  height: 100vh;
  background-color: rgba(0, 0, 0, .9) !important;
  overflow: auto;
}
</style>
