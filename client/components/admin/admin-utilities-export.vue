<template>  
  <v-card>
    <v-toolbar flat color="primary" dark dense>
      <div class="subtitle-1">{{ $t('admin:utilities.exportTitle') }}</div>
    </v-toolbar>
    <v-card-text>
      <div class="text-center"><img class="animated fadeInUp wait-p1s" src="/_assets/svg/icon-big-parcel.svg">
        <div class="body-2">Export to tarball / file system</div>
      </div>
      <v-divider class="my-4"></v-divider>
      <div class="body-2">What do you want to export?</div>
      <v-checkbox v-for="choice of entityChoices" :key="choice.key" :label="choice.label" :value="choice.key" color="deep-orange darken-2" hide-details v-model="entities">
        <template v-slot:label>
          <div><strong class="deep-orange--text text--darken-2">{{choice.label}}</strong>
            <div class="text-caption">{{choice.hint}}</div>
          </div>
        </template>
      </v-checkbox>
      <v-text-field class="mt-7" outlined label="Target Folder Path" hint="Either an absolute path or relative to the Wiki.js installation folder, where exported content will be saved to. Note that the folder MUST be empty!" persistent-hint v-model="filePath"></v-text-field>
      <v-alert class="mt-3" color="deep-orange" outlined icon="mdi-alert" prominent>
        <div class="body-2">Depending on your selection, the archive could contain sensitive data such as site configuration keys and hashed user passwords. Ensure the exported archive is treated accordingly.</div>
        <div class="body-2">For example, you may want to encrypt the archive if stored for backup purposes.</div>
      </v-alert>
    </v-card-text>
    <v-card-chin>
      <v-btn class="px-3 ml-0" depressed color="deep-orange darken-2" :disabled="entities.length < 1" @click="startExport">
        <v-icon left color="white">mdi-database-export</v-icon><span class="white--text">Start Export</span>
      </v-btn>
    </v-card-chin>
    <v-dialog v-model="isLoading" persistent max-width="350">
      <v-card color="deep-orange darken-2" dark>
        <v-card-text class="pa-10 text-center">
          <self-building-square-spinner class="animated fadeIn" :animation-duration="4500" :size="40" color="#FFF" style="margin: 0 auto;"></self-building-square-spinner>
          <div class="mt-5 body-1 white--text">Exporting...</div>
          <div class="caption">Please wait, this may take a while</div>
          <v-progress-linear class="mt-5" color="white" :value="progress" stream rounded :buffer-value="0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-dialog v-model="isSuccess" persistent max-width="350">
      <v-card color="green darken-2" dark>
        <v-card-text class="pa-10 text-center">
          <v-icon size="60">mdi-check-circle-outline</v-icon>
          <div class="my-5 body-1 white--text">Export completed</div>
        </v-card-text>
        <v-card-actions class="green darken-1">
          <v-spacer></v-spacer>
          <v-btn class="px-5" color="white" outlined @click="isSuccess = false">Close</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="isFailed" persistent max-width="800">
      <v-card color="red darken-2" dark>
        <v-toolbar color="red darken-2" dense>
          <v-icon>mdi-alert</v-icon>
          <div class="body-2 pl-3">Export failed</div>
          <v-spacer></v-spacer>
          <v-btn class="px-5" color="white" text @click="isFailed = false">Close</v-btn>
        </v-toolbar>
        <v-card-text class="pa-5 red darken-4 white--text"><span>{{errorMessage}}</span></v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
import { SelfBuildingSquareSpinner } from 'epic-spinners'

import gql from 'graphql-tag'
import _get from 'lodash/get'

export default {
  components: {
    SelfBuildingSquareSpinner
  },
  data() {
    return {
      entities: [],
      filePath: './data/export',
      isLoading: false,
      isSuccess: false,
      isFailed: false,
      errorMessage: '',
      progress: 0
    }
  },
  computed: {
    entityChoices () {
      return [
        {
          key: 'assets',
          label: 'Assets',
          hint: 'Media files such as images, documents, etc.'
        },
        {
          key: 'comments',
          label: 'Comments',
          hint: 'Comments made using the default comment module only.'
        },
        {
          key: 'navigation',
          label: 'Navigation',
          hint: 'Sidebar links when using Static or Custom Navigation.'
        },
        {
          key: 'pages',
          label: 'Pages',
          hint: 'Page content, tags and related metadata.'
        },
        {
          key: 'history',
          label: 'Pages History',
          hint: 'All previous versions of pages and their related metadata.'
        },
        {
          key: 'settings',
          label: 'Settings',
          hint: 'Site configuration and modules settings.'
        },
        {
          key: 'groups',
          label: 'User Groups',
          hint: 'Group permissions and page rules.'
        },
        {
          key: 'users',
          label: 'Users',
          hint: 'Users metadata and their group memberships.'
        }
      ]
    }
  },
  methods: {
    async checkProgress () {
      try {
        const respStatus = await this.$apollo.query({
          query: gql`
            {
              system {
                exportStatus {
                  status
                  progress
                  message
                  startedAt
                }
              }
            }
          `,
          fetchPolicy: 'network-only'
        })
        const respStatusObj = _get(respStatus, 'data.system.exportStatus', {})
        if (!respStatusObj) {
          throw new Error('An unexpected error occured.')
        } else {
          switch (respStatusObj.status) {
            case 'error': {
              throw new Error(respStatusObj.message || 'An unexpected error occured.')
            }
            case 'running': {
              this.progress = respStatusObj.progress || 0
              window.requestAnimationFrame(() => {
                setTimeout(() => {
                  this.checkProgress()
                }, 5000)
              })
              break
            }
            case 'success': {
              this.isLoading = false
              this.isSuccess = true
              break
            }
            default: {
              throw new Error('Invalid export status.')
            }
          }
        }
      } catch (err) {
        this.errorMessage = err.message
        this.isLoading = false
        this.isFailed = true
      }
    },
    async startExport () {
      this.isFailed = false
      this.isSuccess = false
      this.isLoading = true
      this.progress = 0

      setTimeout(async () => {
        try {
          // -> Initiate export
          const respExport = await this.$apollo.mutate({
            mutation: gql`
              mutation (
                $entities: [String]!
                $path: String!
              ) {
                system {
                  export (
                    entities: $entities
                    path: $path
                  ) {
                    responseResult {
                      succeeded
                      message
                    }
                  }
                }
              }
            `,
            variables: {
              entities: this.entities,
              path: this.filePath
            }
          })

          const respExportObj = _get(respExport, 'data.system.export', {})
          if (!_get(respExportObj, 'responseResult.succeeded', false)) {
            this.errorMessage = _get(respExportObj, 'responseResult.message', 'An unexpected error occurred')
            this.isLoading = false
            this.isFailed = true
            return
          }

          // -> Check for progress
          this.checkProgress()
        } catch (err) {
          this.$store.commit('pushGraphError', err)
          this.isLoading = false
        }
      }, 1500)
    }
  }
}
</script>

<style lang='scss'>

</style>
