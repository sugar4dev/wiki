<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-rest-api.svg" alt="API" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('admin:api.title')}}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft">{{$t('admin:api.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <template v-if="enabled">
            <status-indicator class="mr-3" positive pulse></status-indicator>
            <div class="caption green--text animated fadeInLeft">{{$t('admin:api.enabled')}}</div>
          </template>
          <template v-else>
            <status-indicator class="mr-3" negative pulse></status-indicator>
            <div class="caption red--text animated fadeInLeft">{{$t('admin:api.disabled')}}</div>
          </template>
          <v-spacer></v-spacer>
          <v-btn class="mr-3 animated fadeInDown wait-p2s" outlined color="grey" icon @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="mr-3 animated fadeInDown wait-p1s" :color="enabled ? `red` : `green`" depressed @click="globalSwitch" dark :loading="isToggleLoading">
            <v-icon left>mdi-power</v-icon><span v-if="!enabled">{{$t('admin:api.enableButton')}}</span><span v-else>{{$t('admin:api.disableButton')}}</span>
          </v-btn>
          <v-btn class="animated fadeInDown" color="primary" depressed large @click="newKey" dark>
            <v-icon left>mdi-plus</v-icon><span>{{$t('admin:api.newKeyButton')}}</span>
          </v-btn>
        </div>
        <v-card class="mt-3 animated fadeInUp">
          <v-simple-table v-if="keys && keys.length > 0">
            <template v-slot:default>
              <thead>
                <tr class="grey" :class="$vuetify.theme.dark ? `darken-4-d5` : `lighten-5`">
                  <th>{{$t('admin:api.headerName')}}</th>
                  <th>{{$t('admin:api.headerKeyEnding')}}</th>
                  <th>{{$t('admin:api.headerExpiration')}}</th>
                  <th>{{$t('admin:api.headerCreated')}}</th>
                  <th>{{$t('admin:api.headerLastUpdated')}}</th>
                  <th width="100">{{$t('admin:api.headerRevoke')}}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="key of keys" :key="`key-` + key.id">
                  <td><strong :class="key.isRevoked ? `red--text` : ``">{{ key.name }}</strong><em class="caption ml-1 red--text" v-if="key.isRevoked">(revoked)</em></td>
                  <td class="caption">{{ key.keyShort }}</td>
                  <td :style="key.isRevoked ? `text-decoration: line-through;` : ``">{{ key.expiration | moment('LL') }}</td>
                  <td>{{ key.createdAt | moment('calendar') }}</td>
                  <td>{{ key.updatedAt | moment('calendar') }}</td>
                  <td>
                    <v-btn icon @click="revoke(key)" :disabled="key.isRevoked">
                      <v-icon color="error">mdi-cancel</v-icon>
                    </v-btn>
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
          <v-card-text v-else>
            <v-alert class="mb-0" icon="mdi-information" :value="true" outlined color="info">{{$t('admin:api.noKeyInfo')}}</v-alert>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
    <create-api-key v-model="isCreateDialogShown" @refresh="refresh(false)"></create-api-key>
    <v-dialog v-model="isRevokeConfirmDialogShown" max-width="500" persistent>
      <v-card>
        <div class="dialog-header is-red">{{$t('admin:api.revokeConfirm')}}</div>
        <v-card-text class="pa-4">
          <i18next tag="span" path="admin:api.revokeConfirmText"><strong place="name">{{ current.name }}</strong></i18next>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="isRevokeConfirmDialogShown = false" :disabled="revokeLoading">{{$t('common:actions.cancel')}}</v-btn>
          <v-btn color="red" dark @click="revokeConfirm" :loading="revokeLoading">{{$t('admin:api.revoke')}}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'
import { StatusIndicator } from 'vue-status-indicator'

import CreateApiKey from './admin-api-create.vue'

export default {
  components: {
    StatusIndicator,
    CreateApiKey
  },
  data() {
    return {
      enabled: false,
      isToggleLoading: false,
      keys: [],
      isCreateDialogShown: false,
      isRevokeConfirmDialogShown: false,
      revokeLoading: false,
      current: {}
    }
  },
  methods: {
    async refresh (notify = true) {
      this.$apollo.queries.keys.refetch()
      if (notify) {
        this.$store.commit('showNotification', {
          message: this.$t('admin:api.refreshSuccess'),
          style: 'success',
          icon: 'cached'
        })
      }
    },
    async globalSwitch () {
      this.isToggleLoading = true
      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation ($enabled: Boolean!) {
              authentication {
                setApiState (enabled: $enabled) {
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
            enabled: !this.enabled
          },
          watchLoading (isLoading) {
            this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-api-toggle')
          }
        })
        if (_.get(resp, 'data.authentication.setApiState.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.enabled ? this.$t('admin:api.toggleStateDisabledSuccess') : this.$t('admin:api.toggleStateEnabledSuccess'),
            icon: 'check'
          })
          await this.$apollo.queries.enabled.refetch()
        } else {
          this.$store.commit('showNotification', {
            style: 'red',
            message: _.get(resp, 'data.authentication.setApiState.responseResult.message', 'An unexpected error occurred.'),
            icon: 'alert'
          })
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.isToggleLoading = false
    },
    async newKey () {
      this.isCreateDialogShown = true
    },
    revoke (key) {
      this.current = key
      this.isRevokeConfirmDialogShown = true
    },
    async revokeConfirm () {
      this.revokeLoading = true
      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation ($id: Int!) {
              authentication {
                revokeApiKey (id: $id) {
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
            id: this.current.id
          },
          watchLoading (isLoading) {
            this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-api-revoke')
          }
        })
        if (_.get(resp, 'data.authentication.revokeApiKey.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.$t('admin:api.revokeSuccess'),
            icon: 'check'
          })
          this.refresh(false)
        } else {
          this.$store.commit('showNotification', {
            style: 'red',
            message: _.get(resp, 'data.authentication.revokeApiKey.responseResult.message', 'An unexpected error occurred.'),
            icon: 'alert'
          })
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.isRevokeConfirmDialogShown = false
      this.revokeLoading = false
    }
  },
  apollo: {
    enabled: {
      query: gql`
        {
          authentication {
            apiState
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => data.authentication.apiState,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-api-state-refresh')
      }
    },
    keys: {
      query: gql`
        {
          authentication {
            apiKeys {
              id
              name
              keyShort
              expiration
              isRevoked
              createdAt
              updatedAt
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => data.authentication.apiKeys,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-api-keys-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
