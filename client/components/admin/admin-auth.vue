<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-unlock.svg" alt="Authentication" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{ $t('admin:auth.title') }}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p4s">{{ $t('admin:auth.subtitle') }}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p3s" icon outlined color="grey" href="https://docs.requarks.io/auth" target="_blank">
            <v-icon>mdi-help-circle</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown wait-p2s mx-3" icon outlined color="grey" @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown" color="success" @click="save" depressed large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
      </v-flex>
      <v-flex lg3 xs12>
        <v-card class="animated fadeInUp">
          <v-toolbar flat color="teal" dark dense>
            <div class="subtitle-1">{{$t('admin:auth.activeStrategies')}}</div>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <draggable v-model="activeStrategies" handle=".is-handle" direction="vertical">
              <transition-group>
                <v-list-item v-for="(str, idx) in activeStrategies" :key="str.key" @click="selectedStrategy = str.key" :class="selectedStrategy === str.key ? ($vuetify.theme.dark ? `grey darken-5` : `teal lighten-5`) : ``">
                  <v-list-item-avatar class="is-handle" size="24">
                    <v-icon :color="selectedStrategy === str.key ? `teal` : `grey`">mdi-drag-horizontal</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title class="body-2" :class="selectedStrategy === str.key ? `teal--text` : ``">{{ str.displayName }}</v-list-item-title>
                    <v-list-item-subtitle>
                      <div class="caption" :class="selectedStrategy === str.key ? `teal--text ` : ``">{{ str.strategy.title }}</div>
                    </v-list-item-subtitle>
                  </v-list-item-content>
                  <v-list-item-avatar v-if="selectedStrategy === str.key" size="24">
                    <v-icon class="animated fadeInLeft" color="teal" large>mdi-chevron-right</v-icon>
                  </v-list-item-avatar>
                </v-list-item>
              </transition-group>
            </draggable>
          </v-list>
          <v-card-chin>
            <v-menu offset-y bottom min-width="250px" max-width="550px" max-height="50vh" style="flex: 1 1;" center>
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" color="primary" depressed block>
                  <v-icon left>mdi-plus</v-icon><span>{{$t('admin:auth.addStrategy')}}</span>
                </v-btn>
              </template>
              <v-list dense>
                <template v-for="(str, idx) of strategies">
                  <v-list-item :key="str.key" :disabled="str.isDisabled" @click="addStrategy(str)">
                    <v-list-item-avatar height="24" width="48" tile>
                      <v-img :src="str.logo" width="48px" height="24px" contain :style="str.isDisabled ? `opacity: .25;` : ``"></v-img>
                    </v-list-item-avatar>
                    <v-list-item-content>
                      <v-list-item-title>{{str.title}}</v-list-item-title>
                      <v-list-item-subtitle>
                        <div class="caption" :style="str.isDisabled ? `opacity: .4;` : ``">{{str.description}}</div>
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                  <v-divider v-if="idx < strategies.length - 1"></v-divider>
                </template>
              </v-list>
            </v-menu>
          </v-card-chin>
        </v-card>
      </v-flex>
      <v-flex xs12 lg9>
        <v-card class="animated fadeInUp wait-p2s">
          <v-toolbar color="primary" dense flat dark>
            <div class="subtitle-1">{{strategy.displayName}} <em>({{strategy.strategy.title}})</em></div>
            <v-spacer></v-spacer>
            <v-btn small outlined dark color="white" :disabled="strategy.key === `local`" @click="deleteStrategy()">
              <v-icon left>mdi-close</v-icon><span>{{$t('common:actions.delete')}}</span>
            </v-btn>
          </v-toolbar>
          <v-card-info color="blue">
            <div><span>{{strategy.strategy.description}}</span>
              <div class="caption"><a :href="strategy.strategy.website">{{strategy.strategy.website}}</a></div>
            </div>
            <v-spacer></v-spacer>
            <div class="admin-providerlogo"><img :src="strategy.strategy.logo" :alt="strategy.strategy.title"></div>
          </v-card-info>
          <v-card-text>
            <div class="row">
              <div class="col-8">
                <v-text-field outlined :label="$t(`admin:auth.displayName`)" v-model="strategy.displayName" prepend-icon="mdi-format-title" :hint="$t(`admin:auth.displayNameHint`)" persistent-hint></v-text-field>
              </div>
              <div class="col-4">
                <v-switch class="mt-1" :label="$t(`admin:auth.strategyIsEnabled`)" v-model="strategy.isEnabled" color="primary" prepend-icon="mdi-power" :hint="$t(`admin:auth.strategyIsEnabledHint`)" persistent-hint inset :disabled="strategy.key === `local`"></v-switch>
              </div>
            </div>
            <template v-if="strategy.config && Object.keys(strategy.config).length > 0">
              <v-divider></v-divider>
              <div class="overline my-5">{{$t('admin:auth.strategyConfiguration')}}</div>
              <div class="pr-3">
                <template v-for="cfg in strategy.config">
                  <v-select class="mb-3" v-if="cfg.value.type === 'string' && cfg.value.enum" outlined :items="cfg.value.enum" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" :style="cfg.value.maxWidth > 0 ? `max-width:` + cfg.value.maxWidth + `px;` : ``"></v-select>
                  <v-switch class="mb-6" v-else-if="cfg.value.type === 'boolean'" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" color="primary" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint inset></v-switch>
                  <v-textarea class="mb-3" v-else-if="cfg.value.type === 'string' && cfg.value.multiline" outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-textarea>
                  <v-text-field class="mb-3" v-else outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" :style="cfg.value.maxWidth > 0 ? `max-width:` + cfg.value.maxWidth + `px;` : ``"></v-text-field>
                </template>
              </div>
            </template>
            <v-divider></v-divider>
            <div class="overline my-5">{{$t('admin:auth.registration')}}</div>
            <div class="pr-3">
              <v-switch class="ml-3" v-model="strategy.selfRegistration" :label="$t(`admin:auth.selfRegistration`)" color="primary" :hint="$t(`admin:auth.selfRegistrationHint`)" persistent-hint inset></v-switch>
              <v-combobox class="ml-3 mt-5" :label="$t(`admin:auth.domainsWhitelist`)" v-model="strategy.domainWhitelist" prepend-icon="mdi-email-check-outline" outlined :disabled="!strategy.selfRegistration" :hint="$t(`admin:auth.domainsWhitelistHint`)" persistent-hint small-chips deletable-chips clearable multiple chips></v-combobox>
              <v-autocomplete class="mt-3 ml-3" outlined :disabled="!strategy.selfRegistration" :items="groups" item-text="name" item-value="id" :label="$t(`admin:auth.autoEnrollGroups`)" v-model="strategy.autoEnrollGroups" prepend-icon="mdi-account-group" :hint="$t(`admin:auth.autoEnrollGroupsHint`)" small-chips persistent-hint deletable-chips clearable multiple chips></v-autocomplete>
            </div>
          </v-card-text>
        </v-card>
        <v-card class="mt-4 wiki-form animated fadeInUp wait-p4s" v-if="selectedStrategy !== `local`">
          <v-toolbar color="primary" dense flat dark>
            <div class="subtitle-1">{{$t('admin:auth.configReference')}}</div>
          </v-toolbar>
          <v-card-text>
            <div class="body-2">{{$t('admin:auth.configReferenceSubtitle')}}</div>
            <v-alert class="mt-3 radius-7" v-if="host.length < 8" color="red" outlined :value="true" icon="mdi-alert">
              <i18next path="admin:auth.siteUrlNotSetup" tag="span"><strong place="siteUrl">{{$t('admin:general.siteUrl')}}</strong><strong place="general">{{$t('admin:general.title')}}</strong></i18next>
            </v-alert>
            <div class="pa-3 mt-3 radius-7 grey" v-else :class="$vuetify.theme.dark ? `darken-3-d5` : `lighten-3`">
              <div class="body-2"><strong>{{$t('admin:auth.allowedWebOrigins')}}</strong></div>
              <div class="body-2">{{host}}</div>
              <v-divider class="my-3"></v-divider>
              <div class="body-2"><strong>{{$t('admin:auth.callbackUrl')}}</strong></div>
              <div class="body-2">{{host}}/login/{{strategy.key}}/callback</div>
              <v-divider class="my-3"></v-divider>
              <div class="body-2"><strong>{{$t('admin:auth.loginUrl')}}</strong></div>
              <div class="body-2">{{host}}/login</div>
              <v-divider class="my-3"></v-divider>
              <div class="body-2"><strong>{{$t('admin:auth.logoutUrl')}}</strong></div>
              <div class="body-2">{{host}}</div>
              <v-divider class="my-3"></v-divider>
              <div class="body-2"><strong>{{$t('admin:auth.tokenEndpointAuthMethod')}}</strong></div>
              <div class="body-2">HTTP-POST</div>
            </div>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'
import { v4 as uuid } from 'uuid'

import groupsQuery from 'gql/admin/auth/auth-query-groups.gql'
import hostQuery from 'gql/admin/auth/auth-query-host.gql'

import draggable from 'vuedraggable'

export default {
  components: {
    draggable
  },
  filters: {
    startCase(val) { return _.startCase(val) }
  },
  data() {
    return {
      groups: [],
      strategies: [],
      activeStrategies: [],
      selectedStrategy: '',
      host: '',
      strategy: {
        strategy: {}
      }
    }
  },
  watch: {
    selectedStrategy(newValue, oldValue) {
      this.strategy = _.find(this.activeStrategies, ['key', newValue]) || {}
    },
    activeStrategies(newValue, oldValue) {
      this.selectedStrategy = 'local'
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.strategies.refetch()
      await this.$apollo.queries.activeStrategies.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:auth.refreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    addStrategy (str) {
      const newStr = {
        key: uuid(),
        strategy: str,
        config: str.props.map(c => ({
          key: c.key,
          value: {
            ...c,
            value: c.default
          }
        })),
        order: this.activeStrategies.length,
        isEnabled: true,
        displayName: str.title,
        selfRegistration: false,
        domainWhitelist: [],
        autoEnrollGroups: []
      }
      this.activeStrategies = [...this.activeStrategies, newStr]
      this.$nextTick(() => {
        this.selectedStrategy = newStr.key
      })
    },
    deleteStrategy () {
      this.activeStrategies = _.reject(this.activeStrategies, ['key', this.strategy.key])
    },
    async save() {
      this.$store.commit(`loadingStart`, 'admin-auth-savestrategies')
      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation($strategies: [AuthenticationStrategyInput]!) {
              authentication {
                updateStrategies(strategies: $strategies) {
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
            strategies: this.activeStrategies.map((str, idx) => ({
              key: str.key,
              strategyKey: str.strategy.key,
              displayName: str.displayName,
              order: idx,
              isEnabled: str.isEnabled,
              config: str.config.map(cfg => ({...cfg, value: JSON.stringify({ v: cfg.value.value })})),
              selfRegistration: str.selfRegistration,
              domainWhitelist: str.domainWhitelist,
              autoEnrollGroups: str.autoEnrollGroups
            }))
          }
        })
        if (_.get(resp, 'data.authentication.updateStrategies.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            message: this.$t('admin:auth.saveSuccess'),
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(_.get(resp, 'data.authentication.updateStrategies.responseResult.message', this.$t('common:error.unexpected')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-auth-savestrategies')
    }
  },
  apollo: {
    strategies: {
      query: gql`
        query {
          authentication {
            strategies {
              key
              title
              description
              isAvailable
              useForm
              logo
              website
              props {
                key
                value
              }
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.get(data, 'authentication.strategies', []).map(str => ({
        ...str,
        isDisabled: !str.isAvailable || str.key === `local`,
        props: _.sortBy(str.props.map(cfg => ({
          key: cfg.key,
          ...JSON.parse(cfg.value)
        })), [t => t.order])
      })),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-auth-strategies-refresh')
      }
    },
    activeStrategies: {
      query: gql`
        query {
          authentication {
            activeStrategies {
              key
              strategy {
                key
                title
                description
                useForm
                logo
                website
              }
              config {
                key
                value
              }
              order
              isEnabled
              displayName
              selfRegistration
              domainWhitelist
              autoEnrollGroups
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.sortBy(_.get(data, 'authentication.activeStrategies', []).map(str => ({
        ...str,
        config: _.sortBy(str.config.map(cfg => ({
          ...cfg,
          value: JSON.parse(cfg.value)
        })), [t => t.value.order])
      })), ['order']),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-auth-activestrategies-refresh')
      }
    },
    groups: {
      query: groupsQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.groups.list,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-auth-groups-refresh')
      }
    },
    host: {
      query: hostQuery,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.site.config.host),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-auth-host-refresh')
      }
    }
  }
}
</script>
