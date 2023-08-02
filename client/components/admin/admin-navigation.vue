<template>
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-triangle-arrow.svg" alt="Navigation" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('navigation.title')}}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p4s">{{$t('navigation.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p3s" icon outlined color="grey" href="https://docs.requarks.io/navigation" target="_blank">
            <v-icon>mdi-help-circle</v-icon>
          </v-btn>
          <v-btn class="mx-3 animated fadeInDown wait-p2s mr-3" icon outlined color="grey" @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown" color="success" depressed @click="save" large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
        <v-container class="pa-0 mt-3" fluid grid-list-lg>
          <v-row dense>
            <v-col cols="3">
              <v-card class="animated fadeInUp">
                <v-toolbar color="teal" dark dense flat height="56">
                  <v-toolbar-title class="subtitle-1">{{$t('admin:navigation.mode')}}</v-toolbar-title>
                </v-toolbar>
                <v-list nav two-line>
                  <v-list-item-group v-model="config.mode" mandatory :color="$vuetify.theme.dark ? `teal lighten-3` : `teal`">
                    <v-list-item value="TREE">
                      <v-list-item-avatar><img src="/_assets/svg/icon-tree-structure-dotted.svg" alt="Site Tree"></v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{$t('admin:navigation.modeSiteTree.title')}}</v-list-item-title>
                        <v-list-item-subtitle>{{$t('admin:navigation.modeSiteTree.description')}}</v-list-item-subtitle>
                      </v-list-item-content>
                      <v-list-item-avatar>
                        <v-icon v-if="$vuetify.theme.dark" :color="config.mode === `TREE` ? `teal lighten-3` : `grey darken-2`">mdi-check-circle</v-icon>
                        <v-icon v-else :color="config.mode === `TREE` ? `teal` : `grey lighten-3`">mdi-check-circle</v-icon>
                      </v-list-item-avatar>
                    </v-list-item>
                    <v-list-item value="STATIC">
                      <v-list-item-avatar><img src="/_assets/svg/icon-features-list.svg" alt="Static Navigation"></v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{$t('admin:navigation.modeStatic.title')}}</v-list-item-title>
                        <v-list-item-subtitle>{{$t('admin:navigation.modeStatic.description')}}</v-list-item-subtitle>
                      </v-list-item-content>
                      <v-list-item-avatar>
                        <v-icon v-if="$vuetify.theme.dark" :color="config.mode === `STATIC` ? `teal lighten-3` : `grey darken-2`">mdi-check-circle</v-icon>
                        <v-icon v-else :color="config.mode === `STATIC` ? `teal` : `grey lighten-3`">mdi-check-circle</v-icon>
                      </v-list-item-avatar>
                    </v-list-item>
                    <v-list-item value="MIXED">
                      <v-list-item-avatar><img src="/_assets/svg/icon-user-menu-male-dotted.svg" alt="Custom Navigation"></v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{$t('admin:navigation.modeCustom.title')}}</v-list-item-title>
                        <v-list-item-subtitle>{{$t('admin:navigation.modeCustom.description')}}</v-list-item-subtitle>
                      </v-list-item-content>
                      <v-list-item-avatar>
                        <v-icon v-if="$vuetify.theme.dark" :color="config.mode === `MIXED` ? `teal lighten-3` : `grey darken-2`">mdi-check-circle</v-icon>
                        <v-icon v-else :color="config.mode === `MIXED` ? `teal` : `grey lighten-3`">mdi-check-circle</v-icon>
                      </v-list-item-avatar>
                    </v-list-item>
                    <v-list-item value="NONE">
                      <v-list-item-avatar><img src="/_assets/svg/icon-cancel-dotted.svg" alt="None"></v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{$t('admin:navigation.modeNone.title')}}</v-list-item-title>
                        <v-list-item-subtitle>{{$t('admin:navigation.modeNone.description')}}</v-list-item-subtitle>
                      </v-list-item-content>
                      <v-list-item-avatar>
                        <v-icon v-if="$vuetify.theme.dark" :color="config.mode === `none` ? `teal lighten-3` : `grey darken-2`">mdi-check-circle</v-icon>
                        <v-icon v-else :color="config.mode === `none` ? `teal` : `grey lighten-3`">mdi-check-circle</v-icon>
                      </v-list-item-avatar>
                    </v-list-item>
                  </v-list-item-group>
                </v-list>
              </v-card>
            </v-col>
            <v-col cols="9" v-if="config.mode === `MIXED` || config.mode === `STATIC`">
              <v-card class="animated fadeInUp wait-p2s">
                <v-row no-gutters align="stretch">
                  <v-col style="flex: 0 0 350px;">
                    <v-card class="grey" flat style="height: 100%; border-radius: 4px 0 0 4px;" :class="$vuetify.theme.dark ? `darken-4-l5` : `lighten-3`">
                      <div class="teal lighten-1 pa-2 d-flex" style="margin-bottom: 1px; height:56px;">
                        <v-select :disabled="locales.length < 2" label="Locale" hide-details solo flat background-color="teal darken-2" dark dense v-model="currentLang" :items="locales" item-text="nativeName" item-value="code"></v-select>
                        <v-tooltip top>
                          <template v-slot:activator="{ on }">
                            <v-btn class="ml-2" icon tile color="white" v-on="on" @click="copyFromLocaleDialogIsShown = true">
                              <v-icon>mdi-arrange-send-backward</v-icon>
                            </v-btn>
                          </template><span>{{$t('admin:navigation.copyFromLocale')}}</span>
                        </v-tooltip>
                      </div>
                      <v-list class="py-2 blue darken-2" dense nav dark style="border-radius: 0;">
                        <v-list-item v-if="currentTree.length < 1">
                          <v-list-item-avatar size="24">
                            <v-icon color="blue lighten-3">mdi-alert</v-icon>
                          </v-list-item-avatar>
                          <v-list-item-content><em class="caption blue--text text--lighten-4">{{$t('navigation.emptyList')}}</em></v-list-item-content>
                        </v-list-item>
                        <draggable v-model="currentTree">
                          <template v-for="navItem in currentTree">
                            <v-list-item v-if="navItem.kind === 'link'" :key="navItem.id" :class="(navItem === current) ? 'blue' : ''" @click="selectItem(navItem)">
                              <v-list-item-avatar size="24" tile>
                                <v-icon v-if="navItem.icon.match(/fa[a-z] fa-/)" size="19">{{ navItem.icon }}</v-icon>
                                <v-icon v-else>{{ navItem.icon }}</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-title>{{navItem.label}}</v-list-item-title>
                            </v-list-item>
                            <div class="py-2 clickable" v-else-if="navItem.kind === 'divider'" :key="navItem.id" :class="(navItem === current) ? 'blue': ''" @click="selectItem(navItem)">
                              <v-divider></v-divider>
                            </div>
                            <v-subheader class="pl-4 clickable" v-else-if="navItem.kind === 'header'" :key="navItem.id" :class="(navItem === current) ? 'blue': ''" @click="selectItem(navItem)">{{navItem.label}}</v-subheader>
                          </template>
                        </draggable>
                      </v-list>
                      <v-card-chin>
                        <v-menu offset-y bottom min-width="200px" style="flex: 1 1;">
                          <template v-slot:activator="{ on }">
                            <v-btn v-on="on" color="primary" depressed block>
                              <v-icon left>mdi-plus</v-icon><span>{{$t('common:actions.add')}}</span>
                            </v-btn>
                          </template>
                          <v-list>
                            <v-list-item @click="addItem('link')">
                              <v-list-item-avatar size="24">
                                <v-icon>mdi-link</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-title>{{$t('navigation.link')}}</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="addItem('header')">
                              <v-list-item-avatar size="24">
                                <v-icon>mdi-format-title</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-title>{{$t('navigation.header')}}</v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="addItem('divider')">
                              <v-list-item-avatar size="24">
                                <v-icon>mdi-minus</v-icon>
                              </v-list-item-avatar>
                              <v-list-item-title>{{$t('navigation.divider')}}</v-list-item-title>
                            </v-list-item>
                          </v-list>
                        </v-menu>
                      </v-card-chin>
                    </v-card>
                  </v-col>
                  <v-col>
                    <v-card flat style="border-radius: 0 4px 4px 0;">
                      <template v-if="current.kind === 'link'">
                        <v-toolbar height="56" color="teal lighten-1" flat dark>
                          <div class="subtitle-1">{{$t('navigation.edit', { kind: $t('navigation.link') })}}</div>
                          <v-spacer></v-spacer>
                          <v-btn class="px-5" color="white" outlined @click="deleteItem(current)">
                            <v-icon left>mdi-delete</v-icon><span>{{$t('navigation.delete', { kind: $t('navigation.link') })}}</span>
                          </v-btn>
                        </v-toolbar>
                        <v-card-text>
                          <v-text-field outlined :label="$t('navigation.label')" prepend-icon="mdi-format-title" v-model="current.label" counter="255"></v-text-field>
                          <v-text-field outlined :label="$t('navigation.icon')" prepend-icon="mdi-dice-5" v-model="current.icon" hide-details></v-text-field>
                          <div class="caption pt-3 pl-5">The default icon set is <strong>Material Design Icons</strong>. In order to use another icon set, you must first select it in the Theme administration section.</div>
                          <div class="caption pt-3 pl-5"><strong>Material Design Icons</strong></div>
                          <div class="caption pl-5">Refer to the <a href="https://materialdesignicons.com/" target="_blank">Material Design Icons Reference</a> for the list of all possible values. You must prefix all values with <code>mdi-</code>, e.g. <code>mdi-home</code></div>
                          <div class="caption pt-3 pl-5"><strong>Font Awesome 5</strong></div>
                          <div class="caption pl-5">Refer to the <a href="https://fontawesome.com/icons?d=gallery&amp;m=free" target="_blank">Font Awesome 5 Reference</a> for the list of all possible values. You must prefix all values with <code>fas fa-</code>, e.g. <code>fas fa-home</code>. Note that some icons use different prefixes (e.g. <code>fab</code>, <code>fad</code>, <code>fal</code>, <code>far</code>).</div>
                          <div class="caption pt-3 pl-5"><strong>Font Awesome 4</strong></div>
                          <div class="caption pl-5">Refer to the <a href="https://fontawesome.com/v4.7.0/icons/" target="_blank">Font Awesome 4 Reference</a> for the list of all possible values. You must prefix all values with <code>fa fa-</code>, e.g. <code>fa fa-home</code></div>
                        </v-card-text>
                        <v-divider></v-divider>
                        <v-card-text>
                          <v-select outlined :label="$t('navigation.targetType')" prepend-icon="mdi-near-me" :items="navTypes" v-model="current.targetType" hide-details></v-select>
                          <v-text-field class="mt-4" v-if="current.targetType === `external` || current.targetType === `externalblank`" outlined :label="$t('navigation.target')" prepend-icon="mdi-near-me" v-model="current.target" hide-details></v-text-field>
                          <div class="d-flex align-center mt-4" v-else-if="current.targetType === 'page'">
                            <v-btn class="ml-8" color="primary" dark @click="selectPage">
                              <v-icon left>mdi-magnify</v-icon><span>{{$t('admin:navigation.selectPageButton')}}</span>
                            </v-btn>
                            <div class="caption ml-4 primary--text">{{current.target}}</div>
                          </div>
                          <v-text-field v-else-if="current.targetType === `search`" outlined :label="$t('navigation.navType.searchQuery')" prepend-icon="search" v-model="current.target"></v-text-field>
                        </v-card-text>
                        <v-divider></v-divider>
                      </template>
                      <template v-else-if="current.kind === 'header'">
                        <v-toolbar height="56" color="teal lighten-1" flat dark>
                          <div class="subtitle-1">{{$t('navigation.edit', { kind: $t('navigation.header') })}}</div>
                          <v-spacer></v-spacer>
                          <v-btn class="px-5" color="white" outlined @click="deleteItem(current)">
                            <v-icon left>mdi-delete</v-icon><span>{{$t('navigation.delete', { kind: $t('navigation.header') })}}</span>
                          </v-btn>
                        </v-toolbar>
                        <v-card-text>
                          <v-text-field outlined :label="$t('navigation.label')" prepend-icon="mdi-format-title" v-model="current.label"></v-text-field>
                        </v-card-text>
                        <v-divider></v-divider>
                      </template>
                      <div v-else-if="current.kind === 'divider'">
                        <v-toolbar height="56" color="teal lighten-1" flat dark>
                          <div class="subtitle-1">{{$t('navigation.edit', { kind: $t('navigation.divider') })}}</div>
                          <v-spacer></v-spacer>
                          <v-btn class="px-5" color="white" outlined @click="deleteItem(current)">
                            <v-icon left>mdi-delete</v-icon><span>{{$t('navigation.delete', { kind: $t('navigation.divider') })}}</span>
                          </v-btn>
                        </v-toolbar>
                      </div>
                      <v-card-text v-if="current.kind">
                        <v-radio-group class="pl-8" v-model="current.visibilityMode" mandatory hide-details>
                          <v-radio :label="$t('admin:navigation.visibilityMode.all')" value="all" color="primary"></v-radio>
                          <v-radio class="mt-3" :label="$t('admin:navigation.visibilityMode.restricted')" value="restricted" color="primary"></v-radio>
                        </v-radio-group>
                        <div class="pl-8">
                          <v-select class="pl-8 mt-3" item-text="name" item-value="id" outlined prepend-icon="mdi-account-group" label="Groups" :disabled="current.visibilityMode !== `restricted`" v-model="current.visibilityGroups" :items="groups" persistent-hint clearable multiple></v-select>
                        </div>
                      </v-card-text>
                      <template v-else>
                        <v-toolbar height="56" color="teal lighten-1" flat dark></v-toolbar>
                        <v-card-text class="grey--text" v-if="currentTree.length > 0">{{$t('navigation.noSelectionText')}}</v-card-text>
                        <v-card-text class="grey--text" v-else>{{$t('navigation.noItemsText')}}</v-card-text>
                      </template>
                    </v-card>
                  </v-col>
                </v-row>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-flex>
    </v-layout>
    <v-dialog v-model="copyFromLocaleDialogIsShown" max-width="650" persistent>
      <v-card>
        <div class="dialog-header is-short is-teal">
          <v-icon class="mr-3" color="white">mdi-arrange-send-backward</v-icon><span>{{$t('admin:navigation.copyFromLocale')}}</span>
        </div>
        <v-card-text class="pt-5">
          <div class="body-2">{{$t('admin:navigation.copyFromLocaleInfoText')}}</div>
          <v-select class="mt-3" :items="locales" item-text="nativeName" item-value="code" outlined prepend-icon="mdi-web" v-model="copyFromLocaleCode" :label="$t(`admin:navigation.sourceLocale`)" :hint="$t(`admin:navigation.sourceLocaleHint`)" persistent-hint></v-select>
        </v-card-text>
        <v-card-chin>
          <v-spacer></v-spacer>
          <v-btn text @click="copyFromLocaleDialogIsShown = false">{{$t('common:actions.cancel')}}</v-btn>
          <v-btn class="px-3" depressed color="primary" @click="copyFromLocale">
            <v-icon left>mdi-chevron-right</v-icon><span>{{$t('common:actions.copy')}}</span>
          </v-btn>
        </v-card-chin>
      </v-card>
    </v-dialog>
    <page-selector mode="select" v-model="selectPageModal" :open-handler="selectPageHandle" path="home" :locale="currentLang"></page-selector>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'
import { v4 as uuid } from 'uuid'

import groupsQuery from 'gql/admin/users/users-query-groups.gql'

import draggable from 'vuedraggable'

/* global siteConfig, siteLangs */

export default {
  components: {
    draggable
  },
  data() {
    return {
      selectPageModal: false,
      trees: [],
      current: {},
      currentLang: siteConfig.lang,
      groups: [],
      copyFromLocaleDialogIsShown: false,
      config: {
        mode: 'NONE'
      },
      allLocales: [],
      copyFromLocaleCode: 'en'
    }
  },
  computed: {
    navTypes () {
      return [
        { text: this.$t('navigation.navType.external'), value: 'external' },
        { text: this.$t('navigation.navType.externalblank'), value: 'externalblank' },
        { text: this.$t('navigation.navType.home'), value: 'home' },
        { text: this.$t('navigation.navType.page'), value: 'page' }
        // { text: this.$t('navigation.navType.searchQuery'), value: 'search' }
      ]
    },
    locales () {
      return _.intersectionBy(this.allLocales, _.unionBy(siteLangs, [{ code: 'en' }, { code: siteConfig.lang }], 'code'), 'code')
    },
    currentTree: {
      get () {
        return _.get(_.find(this.trees, ['locale', this.currentLang]), 'items', null) || []
      },
      set (val) {
        const tree = _.find(this.trees, ['locale', this.currentLang])
        if (tree) {
          tree.items = val
        } else {
          this.trees = [...this.trees, {
            locale: this.currentLang,
            items: val
          }]
        }
      }
    }
  },
  watch: {
    currentLang (newValue, oldValue) {
      this.$nextTick(() => {
        if (this.currentTree.length > 0) {
          this.current = this.currentTree[0]
        } else {
          this.current = {}
        }
      })
    }
  },
  methods: {
    addItem(kind) {
      let newItem = {
        id: uuid(),
        kind,
        visibilityMode: 'all',
        visibilityGroups: []
      }
      switch (kind) {
        case 'link':
          newItem = {
            ...newItem,
            label: this.$t('navigation.untitled', { kind: this.$t(`navigation.link`) }),
            icon: 'mdi-chevron-right',
            targetType: 'home',
            target: ''
          }
          break
        case 'header':
          newItem.label = this.$t('navigation.untitled', { kind: this.$t(`navigation.header`) })
          break
      }
      this.currentTree = [...this.currentTree, newItem]
      this.current = newItem
    },
    deleteItem(item) {
      this.currentTree = _.pull(this.currentTree, item)
      this.current = {}
    },
    selectItem(item) {
      this.current = item
    },
    selectPage() {
      this.selectPageModal = true
    },
    selectPageHandle ({ path, locale }) {
      this.current.target = `/${locale}/${path}`
    },
    copyFromLocale () {
      this.copyFromLocaleDialogIsShown = false
      this.currentTree = [...this.currentTree, ..._.get(_.find(this.trees, ['locale', this.copyFromLocaleCode]), 'items', null) || []]
    },
    async save() {
      this.$store.commit(`loadingStart`, 'admin-navigation-save')
      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation ($tree: [NavigationTreeInput]!, $mode: NavigationMode!) {
              navigation{
                updateTree(tree: $tree) {
                  responseResult {
                    succeeded
                    errorCode
                    slug
                    message
                  }
                },
                updateConfig(mode: $mode) {
                  responseResult {
                    succeeded
                    errorCode
                    slug
                    message
                  }
                }
              }
            }
          `,
          variables: {
            tree: this.trees,
            mode: this.config.mode
          }
        })
        if (_.get(resp, 'data.navigation.updateTree.responseResult.succeeded', false) && _.get(resp, 'data.navigation.updateConfig.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            message: this.$t('navigation.saveSuccess'),
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(_.get(resp, 'data.navigation.updateTree.responseResult.message', 'An unexpected error occurred.'))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-navigation-save')
    },
    async refresh() {
      await this.$apollo.queries.trees.refetch()
      this.current = {}
      this.$store.commit('showNotification', {
        message: 'Navigation has been refreshed.',
        style: 'success',
        icon: 'cached'
      })
    }
  },
  apollo: {
    config: {
      query: gql`
        {
          navigation {
            config {
              mode
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.navigation.config),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-navigation-config')
      }
    },
    trees: {
      query: gql`
        {
          navigation {
            tree {
              locale
              items {
                id
                kind
                label
                icon
                targetType
                target
                visibilityMode
                visibilityGroups
              }
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.navigation.tree),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-navigation-tree')
      }
    },
    groups: {
      query: groupsQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.groups.list,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-navigation-groups')
      }
    },
    allLocales: {
      query: gql`
        {
          localization {
            locales {
              code
              name
              nativeName
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => data.localization.locales,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-navigation-locales')
      }
    }
  }
}
</script>

<style lang='scss' scoped>

.clickable {
  cursor: pointer;

  &:hover {
    background-color: rgba(mc('blue', '500'), .25);
  }
}

</style>
