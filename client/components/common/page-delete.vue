<template>  
  <v-dialog v-model="isShown" max-width="550" persistent overlay-color="red darken-4" overlay-opacity=".7">
    <v-card>
      <div class="dialog-header is-short is-red">
        <v-icon class="mr-2" color="white">mdi-file-document-box-remove-outline</v-icon><span>{{$t('common:page.delete')}}</span>
      </div>
      <v-card-text class="pt-5">
        <i18next class="body-1" path="common:page.deleteTitle" tag="div"><span class="red--text text--darken-2" place="title">{{pageTitle}}</span></i18next>
        <div class="caption">{{$t('common:page.deleteSubtitle')}}</div>
        <v-chip class="mt-3 ml-0 mr-1" label color="red lighten-4" small>
          <div class="caption red--text text--darken-2">{{pageLocale.toUpperCase()}}</div>
        </v-chip>
        <v-chip class="mt-3 mx-0" label color="red lighten-5" small><span class="red--text text--darken-2">/{{pagePath}}</span></v-chip>
      </v-card-text>
      <v-card-chin>
        <v-spacer></v-spacer>
        <v-btn text @click="discard" :disabled="loading">{{$t('common:actions.cancel')}}</v-btn>
        <v-btn class="px-4 white--text" color="red darken-2" @click="deletePage" :loading="loading">{{$t('common:actions.delete')}}</v-btn>
      </v-card-chin>
    </v-card>
  </v-dialog>
</template>

<script>
import _ from 'lodash'
import { get } from 'vuex-pathify'

import deletePageMutation from 'gql/common/common-pages-mutation-delete.gql'

export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      loading: false
    }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    },
    pageTitle: get('page/title'),
    pagePath: get('page/path'),
    pageLocale: get('page/locale'),
    pageId: get('page/id')
  },
  watch: {
    isShown(newValue, oldValue) {
      if (newValue) {
        document.body.classList.add('page-deleted-pending')
      }
    }
  },
  methods: {
    discard() {
      document.body.classList.remove('page-deleted-pending')
      this.isShown = false
    },
    async deletePage() {
      this.loading = true
      this.$store.commit(`loadingStart`, 'page-delete')
      this.$nextTick(async () => {
        try {
          const resp = await this.$apollo.mutate({
            mutation: deletePageMutation,
            variables: {
              id: this.pageId
            }
          })
          if (_.get(resp, 'data.pages.delete.responseResult.succeeded', false)) {
            this.isShown = false
            _.delay(() => {
              document.body.classList.add('page-deleted')
              _.delay(() => {
                window.location.assign('/')
              }, 1200)
            }, 400)
          } else {
            throw new Error(_.get(resp, 'data.pages.delete.responseResult.message', this.$t('common:error.unexpected')))
          }
        } catch (err) {
          this.$store.commit('pushGraphError', err)
        }
        this.$store.commit(`loadingStop`, 'page-delete')
        this.loading = false
      })
    }
  }
}
</script>

<style lang='scss'>
  body.page-deleted-pending {
    perspective: 50vw;
    height: 100vh;
    overflow: hidden;

    .application {
      background-color: mc('grey', '900');
    }
    .application--wrap {
      transform-style: preserve-3d;
      transform: translateZ(-5vw) rotateX(2deg);
      border-radius: 7px;
      overflow: hidden;
    }
  }
  body.page-deleted {
    perspective: 50vw;

    .application--wrap {
      transform-style: preserve-3d;
      transform: translateZ(-1000vw) rotateX(60deg);
      opacity: 0;
    }
  }
</style>
