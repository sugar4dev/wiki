<template>  
  <v-dialog v-model="isShown" max-width="700">
    <v-card>
      <div class="dialog-header is-short is-indigo">
        <v-icon class="mr-2" color="white">mdi-alert</v-icon><span>{{$t('editor:conflict.title')}}</span>
      </div>
      <v-card-text class="pt-4">
        <i18next class="body-2" tag="div" path="editor:conflict.infoGeneric"><strong place="authorName">{{latest.authorName}}</strong><span place="date" :title="$options.filters.moment(latest.updatedAt, `LLL`)">{{ latest.updatedAt | moment('from') }}.</span></i18next>
        <v-btn class="mt-2" outlined color="indigo" small :href="`/` + latest.locale + `/` + latest.path" target="_blank">
          <v-icon left>mdi-open-in-new</v-icon><span>{{$t('editor:conflict.viewLatestVersion')}}</span>
        </v-btn>
        <div class="body-2 mt-5"><strong>{{$t('editor:conflict.whatToDo')}}</strong></div>
        <div class="body-2 mt-1">
          <v-icon color="indigo">mdi-alpha-l-box</v-icon> {{$t('editor:conflict.whatToDoLocal')}}
        </div>
        <div class="body-2 mt-1">
          <v-icon color="indigo">mdi-alpha-r-box</v-icon> {{$t('editor:conflict.whatToDoRemote')}}
        </div>
      </v-card-text>
      <v-card-chin>
        <v-spacer></v-spacer>
        <v-btn text @click="close">{{$t('common:actions.cancel')}}</v-btn>
        <v-btn class="px-4" color="indigo" @click="useLocal" dark :title="$t(`editor:conflict.useLocalHint`)">
          <v-icon left>mdi-alpha-l-box</v-icon><span>{{$t('editor:conflict.useLocal')}}</span>
        </v-btn>
        <v-dialog v-model="isRemoteConfirmDiagShown" width="500">
          <template v-slot:activator="{ on }">
            <v-btn class="ml-3" color="indigo" dark v-on="on" :title="$t(`editor:conflict.useRemoteHint`)">
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
      </v-card-chin>
    </v-card>
  </v-dialog>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'

export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      latest: {
        updatedAt: '',
        authorName: '',
        content: '',
        locale: '',
        path: ''
      },
      isRemoteConfirmDiagShown: false
    }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    }
  },
  methods: {
    close () {
      this.isShown = false
    },
    useLocal () {
      this.$store.set('editor/checkoutDateActive', this.latest.updatedAt)
      this.$root.$emit('resetEditorConflict')
      this.close()
    },
    useRemote () {
      this.$store.set('editor/checkoutDateActive', this.latest.updatedAt)
      this.$store.set('editor/content', this.latest.content)
      this.$root.$emit('overwriteEditorContent')
      this.$root.$emit('resetEditorConflict')
      this.close()
    }
  },
  async mounted () {
    let resp = await this.$apollo.query({
      query: gql`
        query ($id: Int!) {
          pages {
            conflictLatest(id: $id) {
              authorName
              locale
              path
              content
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
  }
}
</script>
