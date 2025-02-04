<template>  
  <v-card class="editor-modal-media animated fadeInLeft" flat tile :class="`is-editor-` + editorKey">
    <v-container class="pa-3" grid-list-lg fluid>
      <v-layout row wrap>
        <v-flex xs12 lg9>
          <v-card class="radius-7 animated fadeInLeft wait-p1s" :light="!$vuetify.theme.dark" :dark="$vuetify.theme.dark">
            <v-card-text>
              <div class="d-flex">
                <v-toolbar class="radius-7" :color="$vuetify.theme.dark ? `teal` : `teal lighten-5`" dense flat height="44">
                  <div class="body-2" :class="$vuetify.theme.dark ? `white--text` : `teal--text`">{{$t('editor:assets.title')}}</div>
                  <v-spacer></v-spacer>
                  <v-btn text icon @click="refresh">
                    <v-icon :color="$vuetify.theme.dark ? `white` : `teal`">mdi-refresh</v-icon>
                  </v-btn>
                </v-toolbar>
                <v-dialog v-model="newFolderDialog" max-width="550">
                  <template v-slot:activator="{ on }">
                    <v-btn class="ml-3 my-0 mr-0 radius-7" outlined large color="teal" :icon="$vuetify.breakpoint.xsOnly" v-on="on">
                      <v-icon :left="$vuetify.breakpoint.mdAndUp">mdi-plus</v-icon><span class="hidden-sm-and-down" :class="$vuetify.theme.dark ? `teal--text text--lighten-3` : ``">{{$t('editor:assets.newFolder')}}</span>
                    </v-btn>
                  </template>
                  <v-card>
                    <div class="dialog-header is-short subtitle-1">{{$t('editor:assets.newFolder')}}</div>
                    <v-card-text class="pt-5">
                      <v-text-field class="md2" outlined prepend-icon="mdi-folder-outline" v-model="newFolderName" :label="$t(`editor:assets.folderName`)" counter="255" @keydown.13="createFolder" @keyup.esc="newFolderDialog = false" ref="folderNameIpt"></v-text-field>
                      <i18next class="caption grey--text text--darken-1 pl-5" path="editor:assets.folderNameNamingRules" tag="div"><a place="namingRules" href="https://docs-beta.requarks.io/guide/assets#naming-restrictions" target="_blank">{{$t('editor:assets.folderNameNamingRulesLink')}}</a></i18next>
                    </v-card-text>
                    <v-card-chin>
                      <v-spacer></v-spacer>
                      <v-btn text @click="newFolderDialog = false">{{$t('common:actions.cancel')}}</v-btn>
                      <v-btn class="px-3" color="primary" @click="createFolder" :disabled="!isFolderNameValid" :loading="newFolderLoading">{{$t('common:actions.create')}}</v-btn>
                    </v-card-chin>
                  </v-card>
                </v-dialog>
              </div>
              <v-toolbar flat dense :color="$vuetify.theme.dark ? `grey darken-3` : `white`">
                <template v-if="folderTree.length > 0">
                  <div class="body-2"><span class="mr-1">/</span>
                    <template v-for="folder of folderTree"><span :key="folder.id">{{folder.name}}</span><span class="mx-1">/</span></template>
                  </div>
                </template>
                <div class="body-2" v-else>/ <em>root</em></div>
              </v-toolbar>
              <template v-if="folders.length > 0 || currentFolderId > 0">
                <v-btn class="is-icon mx-1" :color="$vuetify.theme.dark ? `grey lighten-1` : `grey darken-2`" outlined :dark="currentFolderId > 0" @click="upFolder()" :disabled="currentFolderId === 0">
                  <v-icon>mdi-folder-upload</v-icon>
                </v-btn>
                <v-btn class="btn-normalcase mx-1" v-for="folder of folders" :key="folder.id" depressed color="grey darken-2" dark @click="downFolder(folder)">
                  <v-icon left>mdi-folder</v-icon><span class="caption" style="text-transform: none;">{{ folder.name }}</span>
                </v-btn>
                <v-divider class="mt-2"></v-divider>
              </template>
              <v-data-table :items="assets" :headers="headers" :page.sync="pagination" :items-per-page="15" :loading="loading" must-sort sort-by="ID" sort-desc hide-default-footer dense>
                <template slot="item" slot-scope="props">
                  <tr class="is-clickable" @click.left="currentFileId = props.item.id" @click.right.prevent="" :class="currentFileId === props.item.id ? ($vuetify.theme.dark ? `grey darken-3-d5` : `teal lighten-5`) : ``">
                    <td class="caption" v-if="$vuetify.breakpoint.smAndUp">{{ props.item.id }}</td>
                    <td>
                      <div class="body-2"><strong :class="currentFileId === props.item.id ? `teal--text` : ``">{{ props.item.filename }}</strong></div>
                      <div class="caption grey--text">{{ props.item.description }}</div>
                    </td>
                    <td class="text-xs-center" v-if="$vuetify.breakpoint.lgAndUp">
                      <v-chip class="ma-0" x-small :color="$vuetify.theme.dark ? `grey darken-4` : `grey lighten-4`">
                        <div class="overline">{{props.item.ext.toUpperCase().substring(1)}}</div>
                      </v-chip>
                    </td>
                    <td class="caption" v-if="$vuetify.breakpoint.mdAndUp">{{ props.item.fileSize | prettyBytes }}</td>
                    <td class="caption" v-if="$vuetify.breakpoint.mdAndUp">{{ props.item.createdAt | moment('from') }}</td>
                    <td v-if="$vuetify.breakpoint.smAndUp">
                      <v-menu offset-x min-width="200">
                        <template v-slot:activator="{ on }">
                          <v-btn icon v-on="on" tile small @click.left="currentFileId = props.item.id">
                            <v-icon color="grey darken-2">mdi-dots-horizontal</v-icon>
                          </v-btn>
                        </template>
                        <v-list nav style="border-top: 5px solid #444;">
                          <v-list-item @click="" disabled>
                            <v-list-item-avatar size="24">
                              <v-icon color="teal">mdi-text-short</v-icon>
                            </v-list-item-avatar>
                            <v-list-item-content>{{$t('common:actions.properties')}}</v-list-item-content>
                          </v-list-item>
                          <template v-if="props.item.kind === `IMAGE`">
                            <v-list-item @click="previewDialog = true" disabled>
                              <v-list-item-avatar size="24">
                                <v-icon color="green">mdi-image-search-outline</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-content>{{$t('common:actions.preview')}}</v-list-item-content>
                            </v-list-item>
                            <v-list-item @click="" disabled>
                              <v-list-item-avatar size="24">
                                <v-icon color="indigo">mdi-crop-rotate</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-content>{{$t('common:actions.edit')}}</v-list-item-content>
                            </v-list-item>
                            <v-list-item @click="" disabled>
                              <v-list-item-avatar size="24">
                                <v-icon color="purple">mdi-flash-circle</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-content>{{$t('common:actions.optimize')}}</v-list-item-content>
                            </v-list-item>
                          </template>
                          <v-list-item @click="openRenameDialog">
                            <v-list-item-avatar size="24">
                              <v-icon color="orange">mdi-keyboard-outline</v-icon>
                            </v-list-item-avatar>
                            <v-list-item-content>{{$t('common:actions.rename')}}</v-list-item-content>
                          </v-list-item>
                          <v-list-item @click="" disabled>
                            <v-list-item-avatar size="24">
                              <v-icon color="blue">mdi-file-move</v-icon>
                            </v-list-item-avatar>
                            <v-list-item-content>{{$t('common:actions.move')}}</v-list-item-content>
                          </v-list-item>
                          <v-list-item @click="deleteDialog = true">
                            <v-list-item-avatar size="24">
                              <v-icon color="red">mdi-file-hidden</v-icon>
                            </v-list-item-avatar>
                            <v-list-item-content>{{$t('common:actions.delete')}}</v-list-item-content>
                          </v-list-item>
                        </v-list>
                      </v-menu>
                    </td>
                  </tr>
                </template>
                <template slot="no-data">
                  <v-alert class="mt-3 radius-7" icon="mdi-folder-open-outline" :value="true" outlined color="teal">{{$t('editor:assets.folderEmpty')}}</v-alert>
                </template>
              </v-data-table>
              <div class="text-xs-center py-2" v-if="this.pageTotal > 1">
                <v-pagination v-model="pagination" :length="pageTotal" color="teal"></v-pagination>
              </div>
              <div class="d-flex mt-3">
                <v-toolbar class="radius-7" flat :color="$vuetify.theme.dark ? `grey darken-2` : `grey lighten-4`" dense height="44">
                  <div class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-1` : `grey--text text--darken-1`">{{$t('editor:assets.fileCount', { count: assets.length })}}</div>
                </v-toolbar>
                <v-btn class="ml-3 mr-0 my-0 radius-7" color="red darken-2" large @click="cancel" dark>
                  <v-icon left>mdi-close</v-icon><span>{{$t('common:actions.cancel')}}</span>
                </v-btn>
                <v-btn class="ml-3 mr-0 my-0 radius-7" color="teal" large @click="insert" :disabled="!currentFileId" :dark="currentFileId !== null">
                  <v-icon left>mdi-playlist-plus</v-icon><span>{{$t('common:actions.insert')}}</span>
                </v-btn>
              </div>
            </v-card-text>
          </v-card>
        </v-flex>
        <v-flex xs12 lg3>
          <v-card class="radius-7 animated fadeInRight wait-p3s" :light="!$vuetify.theme.dark" :dark="$vuetify.theme.dark">
            <v-card-text>
              <div class="d-flex">
                <v-toolbar class="radius-7" :color="$vuetify.theme.dark ? `teal` : `teal lighten-5`" dense flat height="44">
                  <v-icon class="mr-3" :color="$vuetify.theme.dark ? `white` : `teal`">mdi-cloud-upload</v-icon>
                  <div class="body-2" :class="$vuetify.theme.dark ? `white--text` : `teal--text`">{{$t('editor:assets.uploadAssets')}}</div>
                </v-toolbar>
                <v-btn class="my-0 ml-3 mr-0 radius-7" outlined large color="teal" @click="browse" v-if="$vuetify.breakpoint.mdAndUp">
                  <v-icon left>mdi-plus-box-multiple</v-icon><span :class="$vuetify.theme.dark ? `teal--text text--lighten-3` : ``">{{$t('common:actions.browse')}}</span>
                </v-btn>
              </div>
              <file-pond class="mt-3" name="mediaUpload" ref="pond" :label-idle="$t(`editor:assets.uploadAssetsDropZone`)" allow-multiple="true" :files="files" max-files="10" :server="filePondServerOpts" :instant-upload="false" :allow-revert="false" @processfile="onFileProcessed"></file-pond>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions class="pa-3">
              <div class="caption grey--text text-darken-2">Max 10 files, 5 MB each</div>
              <v-spacer></v-spacer>
              <v-btn class="px-4" color="teal" dark @click="upload">{{$t('common:actions.upload')}}</v-btn>
            </v-card-actions>
          </v-card>
          <v-card class="mt-3 radius-7 animated fadeInRight wait-p4s" :light="!$vuetify.theme.dark" :dark="$vuetify.theme.dark">
            <v-card-text class="pb-0">
              <v-toolbar class="radius-7" :color="$vuetify.theme.dark ? `teal` : `teal lighten-5`" dense flat>
                <v-icon class="mr-3" :color="$vuetify.theme.dark ? `white` : `teal`">mdi-cloud-download</v-icon>
                <div class="body-2" :class="$vuetify.theme.dark ? `white--text` : `teal--text`">{{$t('editor:assets.fetchImage')}}</div>
                <v-spacer></v-spacer>
                <v-chip class="teal--text" label color="white" small>coming soon</v-chip>
              </v-toolbar>
              <v-text-field class="mt-3" v-model="remoteImageUrl" outlined color="teal" single-line placeholder="https://example.com/image.jpg"></v-text-field>
            </v-card-text>
            <v-divider></v-divider>
            <v-card-actions class="pa-3">
              <div class="caption grey--text text-darken-2">Max 5 MB</div>
              <v-spacer></v-spacer>
              <v-btn class="px-4" color="teal" disabled>{{$t('common:actions.fetch')}}</v-btn>
            </v-card-actions>
          </v-card>
          <v-card class="mt-3 radius-7 animated fadeInRight wait-p4s" :light="!$vuetify.theme.dark" :dark="$vuetify.theme.dark">
            <v-card-text class="pb-0">
              <v-toolbar class="radius-7" :color="$vuetify.theme.dark ? `teal` : `teal lighten-5`" dense flat>
                <v-icon class="mr-3" :color="$vuetify.theme.dark ? `white` : `teal`">mdi-format-align-top</v-icon>
                <div class="body-2" :class="$vuetify.theme.dark ? `white--text` : `teal--text`">{{$t('editor:assets.imageAlign')}}</div>
              </v-toolbar>
              <v-select class="mt-3" v-model="imageAlignment" :items="imageAlignments" outlined single-line color="teal" placeholder="None"></v-select>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
    <v-dialog v-model="renameDialog" max-width="550" persistent>
      <v-card>
        <div class="dialog-header is-short is-orange">
          <v-icon class="mr-2" color="white">mdi-keyboard</v-icon><span>{{$t('editor:assets.renameAsset')}}</span>
        </div>
        <v-card-text class="pt-5">
          <div class="body-2">{{$t('editor:assets.renameAssetSubtitle')}}</div>
          <v-text-field outlined single-line :counter="255" v-model="renameAssetName" @keydown.13="renameAsset" :disabled="renameAssetLoading"></v-text-field>
        </v-card-text>
        <v-card-chin>
          <v-spacer></v-spacer>
          <v-btn text @click="renameDialog = false" :disabled="renameAssetLoading">{{$t('common:actions.cancel')}}</v-btn>
          <v-btn class="px-3 white--text" color="orange darken-3" @click="renameAsset" :loading="renameAssetLoading">{{$t('common:actions.rename')}}</v-btn>
        </v-card-chin>
      </v-card>
    </v-dialog>
    <v-dialog v-model="deleteDialog" max-width="550" persistent>
      <v-card>
        <div class="dialog-header is-short is-red">
          <v-icon class="mr-2" color="white">mdi-trash-can-outline</v-icon><span>{{$t('editor:assets.deleteAsset')}}</span>
        </div>
        <v-card-text class="pt-5">
          <div class="body-2">{{$t('editor:assets.deleteAssetConfirm')}}</div>
          <div class="body-2 red--text text--darken-2">{{currentAsset.filename}}?</div>
          <div class="caption mt-3">{{$t('editor:assets.deleteAssetWarn')}}</div>
        </v-card-text>
        <v-card-chin>
          <v-spacer></v-spacer>
          <v-btn text @click="deleteDialog = false" :disabled="deleteAssetLoading">{{$t('common:actions.cancel')}}</v-btn>
          <v-btn class="px-3 white--text" color="red darken-2" @click="deleteAsset" :loading="deleteAssetLoading">{{$t('common:actions.delete')}}</v-btn>
        </v-card-chin>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
import _ from 'lodash'
import { get, sync } from 'vuex-pathify'
import Cookies from 'js-cookie'
import vueFilePond from 'vue-filepond'
import 'filepond/dist/filepond.min.css'

import listAssetQuery from 'gql/editor/editor-media-query-list.gql'
import listFolderAssetQuery from 'gql/editor/editor-media-query-folder-list.gql'
import createAssetFolderMutation from 'gql/editor/editor-media-mutation-folder-create.gql'
import renameAssetMutation from 'gql/editor/editor-media-mutation-asset-rename.gql'
import deleteAssetMutation from 'gql/editor/editor-media-mutation-asset-delete.gql'

const FilePond = vueFilePond()
const localeSegmentRegex = /^[A-Z]{2}(-[A-Z]{2})?$/i
const disallowedFolderChars = /[A-Z()=.!@#$%?&*+`~<>,;:\\/[\]¬{| ]/

export default {
  components: {
    FilePond
  },
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      folders: [],
      files: [],
      assets: [],
      pagination: 1,
      remoteImageUrl: '',
      imageAlignments: [
        { text: 'None', value: '' },
        { text: 'Left', value: 'left' },
        { text: 'Centered', value: 'center' },
        { text: 'Right', value: 'right' },
        { text: 'Absolute Top Right', value: 'abstopright' }
      ],
      imageAlignment: '',
      loading: false,
      newFolderDialog: false,
      newFolderName: '',
      newFolderLoading: false,
      previewDialog: false,
      renameDialog: false,
      renameAssetName: '',
      renameAssetLoading: false,
      deleteDialog: false,
      deleteAssetLoading: false
    }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    },
    editorKey: get('editor/editorKey'),
    activeModal: sync('editor/activeModal'),
    folderTree: get('editor/media@folderTree'),
    currentFolderId: sync('editor/media@currentFolderId'),
    currentFileId: sync('editor/media@currentFileId'),
    pageTotal () {
      if (!this.assets) {
        return 0
      }

      return Math.ceil(this.assets.length / 15)
    },
    headers() {
      return _.compact([
        this.$vuetify.breakpoint.smAndUp && { text: this.$t('editor:assets.headerId'), value: 'id', width: 80 },
        { text: this.$t('editor:assets.headerFilename'), value: 'filename' },
        this.$vuetify.breakpoint.lgAndUp && { text: this.$t('editor:assets.headerType'), value: 'ext', width: 90 },
        this.$vuetify.breakpoint.mdAndUp && { text: this.$t('editor:assets.headerFileSize'), value: 'fileSize', width: 110 },
        this.$vuetify.breakpoint.mdAndUp && { text: this.$t('editor:assets.headerAdded'), value: 'createdAt', width: 175 },
        this.$vuetify.breakpoint.smAndUp && { text: this.$t('editor:assets.headerActions'), value: '', width: 80, sortable: false, align: 'right' }
      ])
    },
    isFolderNameValid() {
      return this.newFolderName.length > 1 && !localeSegmentRegex.test(this.newFolderName) && !disallowedFolderChars.test(this.newFolderName)
    },
    currentAsset () {
      return _.find(this.assets, ['id', this.currentFileId]) || {}
    },
    filePondServerOpts () {
      const jwtToken = Cookies.get('jwt')
      return {
        process: {
          url: '/u',
          headers: {
            'Authorization': `Bearer ${jwtToken}`
          }
        }
      }
    }
  },
  watch: {
    newFolderDialog(newValue, oldValue) {
      if (newValue) {
        this.$nextTick(() => {
          this.$refs.folderNameIpt.focus()
        })
      }
    }
  },
  filters: {
    prettyBytes(num) {
      if (typeof num !== 'number' || isNaN(num)) {
        throw new TypeError('Expected a number')
      }

      let exponent
      let unit
      let neg = num < 0
      let units = ['B', 'kB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB']

      if (neg) {
        num = -num
      }
      if (num < 1) {
        return (neg ? '-' : '') + num + ' B'
      }
      exponent = Math.min(Math.floor(Math.log(num) / Math.log(1000)), units.length - 1)
      num = (num / Math.pow(1000, exponent)).toFixed(2) * 1
      unit = units[exponent]

      return (neg ? '-' : '') + num + ' ' + unit
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.assets.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('editor:assets.refreshSuccess'),
        style: 'success',
        icon: 'check'
      })
    },
    insert () {
      const asset = _.find(this.assets, ['id', this.currentFileId])
      const assetPath = this.folderTree.map(f => f.slug).join('/')
      this.$root.$emit('editorInsert', {
        kind: asset.kind,
        path: this.currentFolderId > 0 ? `/${assetPath}/${asset.filename}` : `/${asset.filename}`,
        text: asset.filename,
        align: this.imageAlignment
      })
      this.activeModal = ''
    },
    browse () {
      this.$refs.pond.browse()
    },
    async upload () {
      const files = this.$refs.pond.getFiles()
      if (files.length < 1) {
        return this.$store.commit('showNotification', {
          message: this.$t('editor:assets.noUploadError'),
          style: 'warning',
          icon: 'warning'
        })
      }
      for (let file of files) {
        file.setMetadata({
          folderId: this.currentFolderId
        })
      }
      await this.$refs.pond.processFiles()
    },
    async onFileProcessed (err, file) {
      if (err) {
        return this.$store.commit('showNotification', {
          message: this.$t('editor:assets.uploadFailed'),
          style: 'error',
          icon: 'error'
        })
      }
      _.delay(() => {
        this.$refs.pond.removeFile(file.id)
      }, 5000)

      await this.$apollo.queries.assets.refetch()
    },
    downFolder(folder) {
      this.$store.commit('editor/pushMediaFolderTree', folder)
      this.currentFolderId = folder.id
      this.currentFileId = null
    },
    upFolder() {
      this.$store.commit('editor/popMediaFolderTree')
      const parentFolder = _.last(this.folderTree)
      this.currentFolderId = parentFolder ? parentFolder.id : 0
      this.currentFileId = null
    },
    async createFolder() {
      this.$store.commit(`loadingStart`, 'editor-media-createfolder')
      this.newFolderLoading = true
      try {
        const resp = await this.$apollo.mutate({
          mutation: createAssetFolderMutation,
          variables: {
            parentFolderId: this.currentFolderId,
            slug: this.newFolderName
          }
        })
        if (_.get(resp, 'data.assets.createFolder.responseResult.succeeded', false)) {
          await this.$apollo.queries.folders.refetch()
          this.$store.commit('showNotification', {
            message: this.$t('editor:assets.folderCreateSuccess'),
            style: 'success',
            icon: 'check'
          })
          this.newFolderDialog = false
          this.newFolderName = ''
        } else {
          this.$store.commit('pushGraphError', new Error(_.get(resp, 'data.assets.createFolder.responseResult.message')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.newFolderLoading = false
      this.$store.commit(`loadingStop`, 'editor-media-createfolder')
    },
    openRenameDialog() {
      this.renameAssetName = this.currentAsset.filename
      this.renameDialog = true
    },
    async renameAsset() {
      this.$store.commit(`loadingStart`, 'editor-media-renameasset')
      this.renameAssetLoading = true
      try {
        const resp = await this.$apollo.mutate({
          mutation: renameAssetMutation,
          variables: {
            id: this.currentFileId,
            filename: this.renameAssetName
          }
        })
        if (_.get(resp, 'data.assets.renameAsset.responseResult.succeeded', false)) {
          await this.$apollo.queries.assets.refetch()
          this.$store.commit('showNotification', {
            message: this.$t('editor:assets.renameSuccess'),
            style: 'success',
            icon: 'check'
          })
          this.renameDialog = false
          this.renameAssetName = ''
        } else {
          this.$store.commit('pushGraphError', new Error(_.get(resp, 'data.assets.renameAsset.responseResult.message')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.renameAssetLoading = false
      this.$store.commit(`loadingStop`, 'editor-media-renameasset')
    },
    async deleteAsset() {
      this.$store.commit(`loadingStart`, 'editor-media-deleteasset')
      this.deleteAssetLoading = true
      try {
        const resp = await this.$apollo.mutate({
          mutation: deleteAssetMutation,
          variables: {
            id: this.currentFileId
          }
        })
        if (_.get(resp, 'data.assets.deleteAsset.responseResult.succeeded', false)) {
          this.currentFileId = null
          await this.$apollo.queries.assets.refetch()
          this.$store.commit('showNotification', {
            message: this.$t('editor:assets.deleteSuccess'),
            style: 'success',
            icon: 'check'
          })
          this.deleteDialog = false
        } else {
          this.$store.commit('pushGraphError', new Error(_.get(resp, 'data.assets.deleteAsset.responseResult.message')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.deleteAssetLoading = false
      this.$store.commit(`loadingStop`, 'editor-media-deleteasset')
    },
    cancel () {
      this.activeModal = ''
    }
  },
  apollo: {
    folders: {
      query: listFolderAssetQuery,
      variables() {
        return {
          parentFolderId: this.currentFolderId
        }
      },
      fetchPolicy: 'network-only',
      update: (data) => data.assets.folders,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'editor-media-folders-list-refresh')
      }
    },
    assets: {
      query: listAssetQuery,
      variables() {
        return {
          folderId: this.currentFolderId,
          kind: 'ALL'
        }
      },
      throttle: 1000,
      fetchPolicy: 'network-only',
      update: (data) => data.assets.list,
      watchLoading (isLoading) {
        this.loading = isLoading
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'editor-media-list-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>
.editor-modal-media {
  position: fixed !important;
  top: 112px;
  left: 64px;
  z-index: 10;
  width: calc(100vw - 64px - 17px);
  height: calc(100vh - 112px - 24px);
  background-color: rgba(darken(mc('grey', '900'), 3%), .9) !important;
  overflow: auto;

  @include until($tablet) {
    left: 40px;
    width: calc(100vw - 40px);
    height: calc(100vh - 112px - 24px);
  }

  &.is-editor-ckeditor {
    top: 64px;
    left: 0;
    width: 100%;
    height: calc(100vh - 64px - 26px);

    @include until($tablet) {
      top: 56px;
      left: 0;
      width: 100%;
      height: calc(100vh - 56px - 24px);
    }
  }

  &.is-editor-code {
    top: 64px;
    height: calc(100vh - 64px - 26px);

    @include until($tablet) {
      top: 56px;
      height: calc(100vh - 56px - 24px);
    }
  }

  &.is-editor-common {
    top: 64px;
    left: 0;
    width: 100%;
    height: calc(100vh - 64px);

    @include until($tablet) {
      top: 56px;
      left: 0;
      width: 100%;
      height: calc(100vh - 56px);
    }
  }

  .filepond--root {
    margin-bottom: 0;
  }

  .filepond--drop-label {
    cursor: pointer;

    > label {
      cursor: pointer;
    }
  }

  .filepond--file-action-button.filepond--action-process-item {
    display: none;
  }

  .v-btn--icon {
    padding: 0 20px;
  }
}
</style>
