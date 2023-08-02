<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap v-if="page.id">
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-view-details.svg" alt="Edit Page" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline blue--text text--darken-2 animated fadeInLeft">Page Details</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">
              <v-chip class="ml-0 mr-2 caption" label small>ID {{page.id}}</v-chip><span>/{{page.locale}}/{{page.path}}</span>
            </div>
          </div>
          <v-spacer></v-spacer>
          <template v-if="page.isPublished">
            <status-indicator class="mr-3" positive pulse></status-indicator>
            <div class="caption green--text">{{$t('common:page.published')}}</div>
          </template>
          <template v-else>
            <status-indicator class="mr-3" negative pulse></status-indicator>
            <div class="caption red--text">{{$t('common:page.unpublished')}}</div>
          </template>
          <template v-if="page.isPrivate">
            <status-indicator class="mr-3 ml-4" intermediary pulse></status-indicator>
            <div class="caption deep-orange--text">{{$t('common:page.private')}}</div>
          </template>
          <template v-else>
            <status-indicator class="mr-3 ml-4" active pulse></status-indicator>
            <div class="caption blue--text">{{$t('common:page.global')}}</div>
          </template>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p3s" color="grey" icon outlined to="/pages">
            <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <v-menu offset-y origin="top right">
            <template v-slot:activator="{ on }">
              <v-btn class="mx-3 animated fadeInDown wait-p2s" color="black" v-on="on" depressed dark><span>Actions</span>
                <v-icon right>mdi-chevron-down</v-icon>
              </v-btn>
            </template>
            <v-list dense nav>
              <v-list-item :href="`/` + page.locale + `/` + page.path">
                <v-list-item-icon>
                  <v-icon color="indigo">mdi-text-subject</v-icon>
                </v-list-item-icon>
                <v-list-item-title>View</v-list-item-title>
              </v-list-item>
              <v-list-item :href="`/e/` + page.locale + `/` + page.path">
                <v-list-item-icon>
                  <v-icon color="indigo">mdi-pencil</v-icon>
                </v-list-item-icon>
                <v-list-item-title>Edit</v-list-item-title>
              </v-list-item>
              <v-list-item @click="" disabled>
                <v-list-item-icon>
                  <v-icon color="grey">mdi-cube-scan</v-icon>
                </v-list-item-icon>
                <v-list-item-title>Re-Render</v-list-item-title>
              </v-list-item>
              <v-list-item @click="" disabled>
                <v-list-item-icon>
                  <v-icon color="grey">mdi-earth-remove</v-icon>
                </v-list-item-icon>
                <v-list-item-title>Unpublish</v-list-item-title>
              </v-list-item>
              <v-list-item :href="`/s/` + page.locale + `/` + page.path">
                <v-list-item-icon>
                  <v-icon color="indigo">mdi-code-tags</v-icon>
                </v-list-item-icon>
                <v-list-item-title>View Source</v-list-item-title>
              </v-list-item>
              <v-list-item :href="`/h/` + page.locale + `/` + page.path">
                <v-list-item-icon>
                  <v-icon color="indigo">mdi-history</v-icon>
                </v-list-item-icon>
                <v-list-item-title>View History</v-list-item-title>
              </v-list-item>
              <v-list-item @click="" disabled>
                <v-list-item-icon>
                  <v-icon color="grey">mdi-content-duplicate</v-icon>
                </v-list-item-icon>
                <v-list-item-title>Duplicate</v-list-item-title>
              </v-list-item>
              <v-list-item @click="" disabled>
                <v-list-item-icon>
                  <v-icon color="grey">mdi-content-save-move-outline</v-icon>
                </v-list-item-icon>
                <v-list-item-title>Move / Rename</v-list-item-title>
              </v-list-item>
              <v-dialog v-model="deletePageDialog" max-width="500">
                <template v-slot:activator="{ on }">
                  <v-list-item v-on="on">
                    <v-list-item-icon>
                      <v-icon color="red">mdi-trash-can-outline</v-icon>
                    </v-list-item-icon>
                    <v-list-item-title>Delete</v-list-item-title>
                  </v-list-item>
                </template>
                <v-card>
                  <div class="dialog-header is-short is-red">
                    <v-icon class="mr-2" color="white">mdi-file-document-box-remove-outline</v-icon><span>{{$t('common:page.delete')}}</span>
                  </div>
                  <v-card-text class="pt-5">
                    <i18next class="body-2" path="common:page.deleteTitle" tag="div"><span class="red--text text--darken-2" place="title">{{page.title}}</span></i18next>
                    <div class="caption">{{$t('common:page.deleteSubtitle')}}</div>
                    <v-chip class="mt-3 ml-0 mr-1" label color="red lighten-4" disabled small>
                      <div class="caption red--text text--darken-2">{{page.locale.toUpperCase()}}</div>
                    </v-chip>
                    <v-chip class="mt-3 mx-0" label color="red lighten-5" disabled small><span class="red--text text--darken-2">/{{page.path}}</span></v-chip>
                  </v-card-text>
                  <v-card-chin>
                    <v-spacer></v-spacer>
                    <v-btn text @click="deletePageDialog = false" :disabled="loading">{{$t('common:actions.cancel')}}</v-btn>
                    <v-btn class="white--text" color="red darken-2" @click="deletePage" :loading="loading">{{$t('common:actions.delete')}}</v-btn>
                  </v-card-chin>
                </v-card>
              </v-dialog>
            </v-list>
          </v-menu>
          <v-btn class="animated fadeInDown" color="success" large depressed disabled>
            <v-icon left>mdi-check</v-icon><span>Save Changes</span>
          </v-btn>
        </div>
      </v-flex>
      <v-flex xs12 lg6>
        <v-card class="animated fadeInUp">
          <v-toolbar color="primary" dense dark flat>
            <v-icon class="mr-2">mdi-text-subject</v-icon><span>Properties</span>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Title</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.title }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Description</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.description || '-' }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Locale</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.locale }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Path</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.path }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Editor</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.editor || '?' }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Content Type</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.contentType || '?' }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Page Hash</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.hash }}</v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-card>
      </v-flex>
      <v-flex xs12 lg6>
        <v-card class="animated fadeInUp wait-p2s">
          <v-toolbar color="primary" dense dark flat>
            <v-icon class="mr-2">mdi-account-multiple</v-icon><span>Users</span>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <v-list-item>
              <v-list-item-avatar size="24">
                <v-btn icon :to="`/users/` + page.creatorId">
                  <v-icon color="grey">mdi-account</v-icon>
                </v-btn>
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Creator</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.creatorName }} <em class="caption">({{ page.creatorEmail }})</em></v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-list-item-action-text>{{ page.createdAt | moment('calendar') }}</v-list-item-action-text>
              </v-list-item-action>
            </v-list-item>
            <v-divider></v-divider>
            <v-list-item>
              <v-list-item-avatar size="24">
                <v-btn icon :to="`/users/` + page.authorId">
                  <v-icon color="grey">mdi-account</v-icon>
                </v-btn>
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title>
                  <div class="overline grey--text">Last Editor</div>
                </v-list-item-title>
                <v-list-item-subtitle class="body-2" :class="$vuetify.theme.dark ? `grey--text text--lighten-2` : `grey--text text--darken-3`">{{ page.authorName }} <em class="caption">({{ page.authorEmail }})</em></v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-list-item-action-text>{{ page.updatedAt | moment('calendar') }}</v-list-item-action-text>
              </v-list-item-action>
            </v-list-item>
          </v-list>
        </v-card>
      </v-flex>
    </v-layout>
    <v-layout row align-center v-else>
      <v-progress-circular indeterminate width="2" color="grey"></v-progress-circular>
      <div class="body-2 pl-3 grey--text">{{ $t('common:page.loading') }}</div>
    </v-layout>
  </v-container>
</template>
<script>
import _ from 'lodash'
import { StatusIndicator } from 'vue-status-indicator'

import pageQuery from 'gql/admin/pages/pages-query-single.gql'
import deletePageMutation from 'gql/common/common-pages-mutation-delete.gql'

export default {
  components: {
    StatusIndicator
  },
  data() {
    return {
      deletePageDialog: false,
      page: {},
      loading: false
    }
  },
  methods: {
    async deletePage() {
      this.loading = true
      this.$store.commit(`loadingStart`, 'page-delete')
      try {
        const resp = await this.$apollo.mutate({
          mutation: deletePageMutation,
          variables: {
            id: this.page.id
          }
        })
        if (_.get(resp, 'data.pages.delete.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'green',
            message: `Page deleted successfully.`,
            icon: 'check'
          })
          this.$router.replace('/pages')
        } else {
          throw new Error(_.get(resp, 'data.pages.delete.responseResult.message', this.$t('common:error.unexpected')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'page-delete')
    },
    async rerenderPage() {
      this.$store.commit('showNotification', {
        style: 'indigo',
        message: `Coming soon...`,
        icon: 'directions_boat'
      })
    }
  },
  apollo: {
    page: {
      query: pageQuery,
      variables() {
        return {
          id: _.toSafeInteger(this.$route.params.id)
        }
      },
      fetchPolicy: 'network-only',
      update: (data) => data.pages.single,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-pages-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
