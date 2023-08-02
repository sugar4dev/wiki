<template>  
  <div class="editor-redirect">
    <div class="editor-redirect-main">
      <div class="editor-redirect-editor">
        <v-container class="px-2 pt-1" fluid>
          <v-row dense>
            <v-col cols="12" lg="8" offset-lg="2" xl="6" offset-xl="3">
              <v-card class="pt-2">
                <v-card-text>
                  <div class="pb-1">
                    <div class="subtitle-2 primary--text">When a user reaches this page</div>
                    <div class="caption grey--text text--darken-1">and matches one of these rules...</div>
                  </div>
                  <v-timeline dense>
                    <v-slide-x-reverse-transition group hide-on-leave>
                      <v-timeline-item key="cond-add-new" hide-dot>
                        <v-btn color="primary" @click="">
                          <v-icon left>mdi-plus</v-icon><span>Add Conditional Rule</span>
                        </v-btn>
                      </v-timeline-item>
                      <v-timeline-item key="cond-none" small color="grey">
                        <v-card class="grey lighten-5" flat>
                          <v-card-text>
                            <div class="body-2"><strong>No conditional rule</strong></div><em>Add conditional rules to direct users to a different page based on their group.</em>
                          </v-card-text>
                        </v-card>
                      </v-timeline-item>
                      <v-timeline-item key="cond-rule-1" small color="primary">
                        <v-card class="blue-grey lighten-5" flat>
                          <v-card-text>
                            <div class="d-flex align-center">
                              <div class="body-2"><strong>User is a member of any of these groups:</strong></div>
                              <v-select class="ml-3" color="primary" :items="groups" item-text="name" item-value="id" multiple solo flat hide-details dense chips small-chips></v-select>
                            </div>
                            <v-divider class="my-3"></v-divider>
                            <div class="d-flex align-center">
                              <div class="body-2 mr-3">then redirect to</div>
                              <v-btn-toggle class="mr-3" v-model="fallbackMode" mandatory color="primary" borderless dense>
                                <v-btn class="text-none" value="page">Page</v-btn>
                                <v-btn class="text-none" value="url">External URL</v-btn>
                              </v-btn-toggle>
                              <v-btn class="mr-3" v-if="fallbackMode === `page`" color="primary">
                                <v-icon left>mdi-magnify</v-icon><span>Select Page...</span>
                              </v-btn>
                              <v-text-field v-if="fallbackMode === `url`" label="External URL" outlined hint="Required - Title of the API" hide-details v-model="fallbackUrl" dense single-line></v-text-field>
                            </div>
                          </v-card-text>
                        </v-card>
                      </v-timeline-item>
                    </v-slide-x-reverse-transition>
                  </v-timeline>
                  <v-divider class="mb-5"></v-divider>
                  <div class="subtitle-2 primary--text">Otherwise, redirect to...</div>
                  <div class="caption grey--text text--darken-1 pb-2">This fallback rule is mandatory and used if none of the conditional rules above applies.</div>
                  <div class="d-flex align-center">
                    <v-btn-toggle class="mr-3" v-model="fallbackMode" mandatory color="primary" borderless dense>
                      <v-btn class="text-none" value="page">Page</v-btn>
                      <v-btn class="text-none" value="url">External URL</v-btn>
                    </v-btn-toggle>
                    <v-btn class="mr-3" v-if="fallbackMode === `page`" color="primary">
                      <v-icon left>mdi-magnify</v-icon><span>Select Page...</span>
                    </v-btn>
                    <v-text-field v-if="fallbackMode === `url`" label="External URL" outlined hint="Required - Title of the API" hide-details v-model="fallbackUrl" dense single-line></v-text-field>
                  </div>
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </div>
    </div>
    <v-system-bar class="editor-redirect-sysbar" dark status color="grey darken-3">
      <div class="caption editor-redirect-sysbar-locale">{{locale.toUpperCase()}}</div>
      <div class="caption px-3">/{{path}}</div>
      <template v-if="$vuetify.breakpoint.mdAndUp">
        <v-spacer></v-spacer>
        <div class="caption">Redirect</div>
        <v-spacer></v-spacer>
        <div class="caption">0 rules</div>
      </template>
    </v-system-bar>
  </div>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'
import { v4 as uuid } from 'uuid'
import { get, sync } from 'vuex-pathify'

export default {
  data() {
    return {
      fallbackMode: 'page',
      fallbackUrl: 'https://'
    }
  },
  computed: {
    isMobile() {
      return this.$vuetify.breakpoint.smAndDown
    },
    locale: get('page/locale'),
    path: get('page/path'),
    mode: get('editor/mode'),
    activeModal: sync('editor/activeModal')
  },
  methods: {
  },
  mounted() {
    this.$store.set('editor/editorKey', 'redirect')

    if (this.mode === 'create') {
      this.$store.set('editor/content', '<h1>Title</h1>\n\n<p>Some text here</p>')
    }
  },
  apollo: {
    groups: {
      query: gql`
        {
          groups {
            list {
              id
              name
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => data.groups.list,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'editor-redirect-groups')
      }
    }
  }
}
</script>

<style lang='scss'>
$editor-height: calc(100vh - 64px - 24px);
$editor-height-mobile: calc(100vh - 56px - 16px);

.editor-redirect {
  &-main {
    display: flex;
    width: 100%;
  }

  &-editor {
    background-color: darken(mc('grey', '100'), 4.5%);
    flex: 1 1 50%;
    display: block;
    height: $editor-height;
    position: relative;

    @at-root .theme--dark & {
      background-color: darken(mc('grey', '900'), 4.5%);
    }
  }

  &-sidebar {
    width: 200px;
  }

  &-sysbar {
    padding-left: 0 !important;

    &-locale {
      background-color: rgba(255,255,255,.25);
      display:inline-flex;
      padding: 0 12px;
      height: 24px;
      width: 63px;
      justify-content: center;
      align-items: center;
    }
  }

}
</style>
