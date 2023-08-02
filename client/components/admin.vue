<template>  
  <v-app class="admin">
    <nav-header hide-search>
      <template slot="mid">
        <v-spacer></v-spacer>
        <div class="overline grey--text">{{$t('admin:adminArea')}}</div>
        <v-spacer></v-spacer>
      </template>
    </nav-header>
    <v-navigation-drawer class="pb-0 admin-sidebar" v-model="adminDrawerShown" app fixed clipped :right="$vuetify.rtl" permanent width="300" :class="$vuetify.theme.dark ? `grey darken-4` : ``">
      <vue-scroll :ops="scrollStyle">
        <v-list class="radius-0" dense nav>
          <v-list-item to="/dashboard" color="primary">
            <v-list-item-avatar size="24" tile>
              <v-icon>mdi-view-dashboard-variant</v-icon>
            </v-list-item-avatar>
            <v-list-item-title>{{ $t('admin:dashboard.title') }}</v-list-item-title>
          </v-list-item>
          <template v-if="hasPermission([`manage:system`, `manage:navigation`, `write:pages`, `manage:pages`, `delete:pages`])">
            <v-divider class="my-2"></v-divider>
            <v-subheader class="pl-4">{{ $t('admin:nav.site') }}</v-subheader>
            <v-list-item to="/general" color="primary" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-widgets</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:general.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/locale" color="primary" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-web</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:locale.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/navigation" color="primary" v-if="hasPermission([`manage:system`, `manage:navigation`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-near-me</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:navigation.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/pages" color="primary" v-if="hasPermission([`manage:system`, `write:pages`, `manage:pages`, `delete:pages`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-file-document-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:pages.title') }}</v-list-item-title>
              <v-list-item-action style="min-width:auto;">
                <v-chip x-small :color="$vuetify.theme.dark ? `grey darken-3-d4` : `grey lighten-5`">
                  <div class="caption grey--text">{{ info.pagesTotal }}</div>
                </v-chip>
              </v-list-item-action>
            </v-list-item>
            <v-list-item to="/tags" v-if="hasPermission([`manage:system`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-tag-multiple</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:tags.title') }}</v-list-item-title>
              <v-list-item-action style="min-width:auto;">
                <v-chip x-small :color="$vuetify.theme.dark ? `grey darken-3-d4` : `grey lighten-5`">
                  <div class="caption grey--text">{{ info.tagsTotal }}</div>
                </v-chip>
              </v-list-item-action>
            </v-list-item>
            <v-list-item to="/theme" color="primary" v-if="hasPermission([`manage:system`, `manage:theme`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-palette-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:theme.title') }}</v-list-item-title>
            </v-list-item>
          </template>
          <template v-if="hasPermission([`manage:system`, `manage:groups`, `write:groups`, `manage:users`, `write:users`])">
            <v-divider class="my-2"></v-divider>
            <v-subheader class="pl-4">{{ $t('admin:nav.users') }}</v-subheader>
            <v-list-item to="/groups" color="primary" v-if="hasPermission([`manage:system`, `manage:groups`, `write:groups`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-account-group</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:groups.title') }}</v-list-item-title>
              <v-list-item-action style="min-width:auto;">
                <v-chip x-small :color="$vuetify.theme.dark ? `grey darken-3-d4` : `grey lighten-4`">
                  <div class="caption grey--text">{{ info.groupsTotal }}</div>
                </v-chip>
              </v-list-item-action>
            </v-list-item>
            <v-list-item to="/users" color="primary" v-if="hasPermission([`manage:system`, `manage:groups`, `write:groups`, `manage:users`, `write:users`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-account-box</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:users.title') }}</v-list-item-title>
              <v-list-item-action style="min-width:auto;">
                <v-chip x-small :color="$vuetify.theme.dark ? `grey darken-3-d4` : `grey lighten-4`">
                  <div class="caption grey--text">{{ info.usersTotal }}</div>
                </v-chip>
              </v-list-item-action>
            </v-list-item>
          </template>
          <template v-if="hasPermission(`manage:system`)">
            <v-divider class="my-2"></v-divider>
            <v-subheader class="pl-4">{{ $t('admin:nav.modules') }}</v-subheader>
            <v-list-item to="/analytics" color="primary">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-chart-timeline-variant</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:analytics.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/auth" color="primary">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-lock-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:auth.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/comments">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-comment-text-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:comments.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/rendering" color="primary">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-cogs</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:rendering.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/search" color="primary">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-cloud-search-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:search.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/storage" color="primary">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-harddisk</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:storage.title') }}</v-list-item-title>
            </v-list-item>
          </template>
          <template v-if="hasPermission([`manage:system`, `manage:api`])">
            <v-divider class="my-2"></v-divider>
            <v-subheader class="pl-4">{{ $t('admin:nav.system') }}</v-subheader>
            <v-list-item to="/api" v-if="hasPermission([`manage:system`, `manage:api`])">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-call-split</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:api.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/mail" color="primary" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-email-multiple-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:mail.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/security" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-lock-check</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:security.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/ssl" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-cloud-lock-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:ssl.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/system" color="primary" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-tune</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:system.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-item to="/utilities" color="primary" v-if="hasPermission(`manage:system`)">
              <v-list-item-avatar size="24" tile>
                <v-icon>mdi-wrench-outline</v-icon>
              </v-list-item-avatar>
              <v-list-item-title>{{ $t('admin:utilities.title') }}</v-list-item-title>
            </v-list-item>
            <v-list-group to="/dev" no-action v-if="hasPermission([`manage:system`, `manage:api`])">
              <v-list-item slot="activator">
                <v-list-item-avatar size="24" tile>
                  <v-icon>mdi-dev-to</v-icon>
                </v-list-item-avatar>
                <v-list-item-title>{{ $t('admin:dev.title') }}</v-list-item-title>
              </v-list-item>
              <v-list-item to="/dev-flags" color="primary">
                <v-list-item-title>{{ $t('admin:dev.flags.title') }}</v-list-item-title>
              </v-list-item>
              <v-list-item href="/graphql" color="primary">
                <v-list-item-title>GraphQL</v-list-item-title>
              </v-list-item>
            </v-list-group>
            <v-divider class="my-2"></v-divider>
          </template>
          <v-list-item to="/contribute" color="primary">
            <v-list-item-avatar size="24" tile>
              <v-icon>mdi-heart-outline</v-icon>
            </v-list-item-avatar>
            <v-list-item-title>{{ $t('admin:contribute.title') }}</v-list-item-title>
          </v-list-item>
        </v-list>
      </vue-scroll>
    </v-navigation-drawer>
    <v-main :class="$vuetify.theme.dark ? 'grey darken-5' : 'grey lighten-5'">
      <transition name="admin-router">
        <router-view></router-view>
      </transition>
    </v-main>
    <nav-footer></nav-footer>
    <notify></notify>
    <search-results></search-results>
  </v-app>
</template>

<script>
import _ from 'lodash'
import VueRouter from 'vue-router'
import { get, sync } from 'vuex-pathify'

import statsQuery from 'gql/admin/dashboard/dashboard-query-stats.gql'

import adminStore from '../store/admin'

/* global WIKI */

WIKI.$store.registerModule('admin', adminStore)

const router = new VueRouter({
  mode: 'history',
  base: '/a',
  routes: [
    { path: '/', redirect: '/dashboard' },
    { path: '/dashboard', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-dashboard.vue') },
    { path: '/general', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-general.vue') },
    { path: '/locale', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-locale.vue') },
    { path: '/navigation', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-navigation.vue') },
    { path: '/pages', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-pages.vue') },
    { path: '/pages/:id(\\d+)', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-pages-edit.vue') },
    { path: '/pages/visualize', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-pages-visualize.vue') },
    { path: '/tags', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-tags.vue') },
    { path: '/theme', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-theme.vue') },
    { path: '/groups', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-groups.vue') },
    { path: '/groups/:id(\\d+)', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-groups-edit.vue') },
    { path: '/users', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-users.vue') },
    { path: '/users/:id(\\d+)', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-users-edit.vue') },
    { path: '/analytics', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-analytics.vue') },
    { path: '/auth', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-auth.vue') },
    { path: '/comments', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-comments.vue') },
    { path: '/rendering', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-rendering.vue') },
    { path: '/editor', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-editor.vue') },
    { path: '/extensions', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-extensions.vue') },
    { path: '/logging', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-logging.vue') },
    { path: '/search', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-search.vue') },
    { path: '/storage', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-storage.vue') },
    { path: '/api', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-api.vue') },
    { path: '/mail', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-mail.vue') },
    { path: '/security', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-security.vue') },
    { path: '/ssl', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-ssl.vue') },
    { path: '/system', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-system.vue') },
    { path: '/utilities', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-utilities.vue') },
    { path: '/webhooks', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-webhooks.vue') },
    { path: '/dev-flags', component: () => import(/* webpackChunkName: "admin-dev" */ './admin/admin-dev-flags.vue') },
    { path: '/contribute', component: () => import(/* webpackChunkName: "admin" */ './admin/admin-contribute.vue') }
  ]
})

export default {
  i18nOptions: { namespaces: 'admin' },
  data() {
    return {
      adminDrawerShown: true,
      scrollStyle: {
        vuescroll: {},
        scrollPanel: {
          initialScrollY: 0,
          initialScrollX: 0,
          scrollingX: false,
          easing: 'easeOutQuad',
          speed: 1000,
          verticalNativeBarPos: this.$vuetify.rtl ? `left` : `right`
        },
        rail: {
          gutterOfEnds: '2px'
        },
        bar: {
          onlyShowBarOnScroll: false,
          background: '#CCC',
          hoverStyle: {
            background: '#999'
          }
        }
      }
    }
  },
  computed: {
    info: sync('admin/info'),
    permissions: get('user/permissions')
  },
  router,
  created() {
    this.$store.commit('page/SET_MODE', 'admin')
  },
  methods: {
    hasPermission(prm) {
      if (_.isArray(prm)) {
        return _.some(prm, p => {
          return _.includes(this.permissions, p)
        })
      } else {
        return _.includes(this.permissions, prm)
      }
    }
  },
  apollo: {
    info: {
      query: statsQuery,
      fetchPolicy: 'network-only',
      manual: true,
      result({ data, loading, networkStatus }) {
        this.info = data.system.info
      },
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-stats-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

.admin {
  &.theme--light .application--wrap {
    background-color: lighten(mc('grey', '200'), 2%);
  }
}

.admin-router {
  &-enter-active, &-leave-active {
    transition: opacity .25s ease;
    opacity: 1;
  }
  &-enter-active {
    transition-delay: .25s;
  }
  &-enter, &-leave-to {
    opacity: 0;
  }
}

.admin-sidebar {
  .v-list__tile--active {
    background-color: rgba(mc('theme', 'primary'), .1);

    .v-icon {
      color: mc('theme', 'primary');
    }
  }

  .v-list-group > .v-list-item {
    padding-left: 0;
  }
}

.theme--dark {
  .admin-sidebar .v-list__tile--active {
    background-color: rgba(0,0,0, .2);
    color: mc('blue', '500') !important;

    .v-icon {
      color: mc('blue', '500');
    }
  }
}

.admin-header {
  display: flex;
  justify-content: flex-start;
  align-items: center;

  &-title {
    margin-left: 1rem;
  }
}

.admin-providerlogo {
  width: 250px;
  height: 50px;
  float: right;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin-left: 16px;

  img {
    max-width: 100%;
    max-height: 50px;
  }
}

.v-application.admin {
  code {
    box-shadow: none;
    font-family: 'Roboto Mono', monospace;
    color: mc('pink', '500');
  }
}

</style>
