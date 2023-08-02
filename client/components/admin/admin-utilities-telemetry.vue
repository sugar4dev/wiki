<template>  
  <v-card>
    <v-toolbar flat color="primary" dark dense>
      <div class="subtitle-1">{{ $t('admin:utilities.telemetryTitle') }}</div>
    </v-toolbar>
    <v-form>
      <v-card-text>
        <div class="subtitle-2">What is telemetry?</div>
        <div class="body-2 mt-3">Telemetry allows the developers of Wiki.js to improve the software by collecting basic anonymized data about its usage and the host info. <br> This is entirely optional and <strong>absolutely no</strong> private data (such as content or personal data) is collected.</div>
        <div class="body-2 mt-3">For maximum privacy, a random client ID is generated during setup. This ID is used to group requests together while keeping complete anonymity. You can reset and generate a new one below at any time.</div>
        <v-divider class="my-4"></v-divider>
        <div class="subtitle-2">What is collected?</div>
        <div class="body-2 mt-3">When telemetry is enabled, only the following data is transmitted:</div>
        <v-list>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-information-outline</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title class="body-2">Version of Wiki.js installed</v-list-item-title>
              <v-list-item-subtitle class="caption"><em>e.g. v2.0.123</em></v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-information-outline</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title class="body-2">Basic OS information</v-list-item-title>
              <v-list-item-subtitle class="caption"><em>Platform (Linux, macOS or Windows), Total CPU cores and DB type (PostgreSQL, MySQL, MariaDB, SQLite or SQL Server)</em></v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-information-outline</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title class="body-2">Crash debug data</v-list-item-title>
              <v-list-item-subtitle class="caption"><em>Stack trace of the error</em></v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-information-outline</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title class="body-2">Setup analytics</v-list-item-title>
              <v-list-item-subtitle class="caption"><em>Installation checkpoint reached</em></v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
        <div class="body-2">Note that crash debug data is stored for a maximum of 30 days while analytics are stored for a maximum of 16 months, after which it is permanently deleted.</div>
        <v-divider class="my-4"></v-divider>
        <div class="subtitle-2">What is it used for?</div>
        <div class="body-2 mt-3">Telemetry is used by developers to improve Wiki.js, mostly for the following reasons:</div>
        <v-list dense>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-chevron-right</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title>
                <div class="body-2">Identify critical bugs more easily and fix them in a timely manner.</div>
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-chevron-right</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title>
                <div class="body-2">Understand the upgrade rate of current installations.</div>
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-list-item>
            <v-list-item-avatar>
              <v-icon>mdi-chevron-right</v-icon>
            </v-list-item-avatar>
            <v-list-item-content>
              <v-list-item-title>
                <div class="body-2"> Optimize performance and testing scenarios based on most popular environments.</div>
              </v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
        <div class="body-2">Only authorized developers have access to the data. It is not shared to any 3rd party nor is it used for any other application than improving Wiki.js.</div>
        <v-divider class="my-4"></v-divider>
        <div class="subtitle-2">Settings</div>
        <div class="mt-3">
          <v-switch class="mt-0" v-model="telemetry" label="Enable Telemetry" color="primary" hint="Allow Wiki.js to transmit telemetry data." persistent-hint></v-switch>
        </div>
        <v-divider class="my-4"></v-divider>
        <div class="subtitle-2 mt-3 grey--text text--darken-1">Client ID</div>
        <div class="body-2 mt-2">{{clientId}}</div>
      </v-card-text>
      <v-card-chin>
        <v-btn class="px-3" depressed color="success" @click="updateTelemetry">
          <v-icon left>mdi-chevron-right</v-icon>Save Changes
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn class="px-3" outlined color="grey" @click="resetClientId">
          <v-icon left>mdi-autorenew</v-icon><span>Reset Client ID</span>
        </v-btn>
      </v-card-chin>
    </v-form>
  </v-card>
</template>

<script>
import _ from 'lodash'

import utilityTelemetryResetIdMutation from 'gql/admin/utilities/utilities-mutation-telemetry-resetid.gql'
import utilityTelemetrySetMutation from 'gql/admin/utilities/utilities-mutation-telemetry-set.gql'
import utilityTelemetryQuery from 'gql/admin/utilities/utilities-query-telemetry.gql'

export default {
  data() {
    return {
      telemetry: false,
      clientId: 'N/A'
    }
  },
  methods: {
    async updateTelemetry() {
      this.loading = true
      this.$store.commit(`loadingStart`, 'admin-utilities-telemetry-set')

      try {
        const respRaw = await this.$apollo.mutate({
          mutation: utilityTelemetrySetMutation,
          variables: {
            enabled: this.telemetry
          }
        })
        const resp = _.get(respRaw, 'data.system.setTelemetry.responseResult', {})
        if (resp.succeeded) {
          this.$store.commit('showNotification', {
            message: 'Telemetry updated successfully.',
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }

      this.$store.commit(`loadingStop`, 'admin-utilities-telemetry-set')
      this.loading = false
    },
    async resetClientId() {
      this.loading = true
      this.$store.commit(`loadingStart`, 'admin-utilities-telemetry-resetid')

      try {
        const respRaw = await this.$apollo.mutate({
          mutation: utilityTelemetryResetIdMutation
        })
        const resp = _.get(respRaw, 'data.system.resetTelemetryClientId.responseResult', {})
        if (resp.succeeded) {
          this.$apollo.queries.telemetry.refetch()
          this.$store.commit('showNotification', {
            message: 'Telemetry Client ID reset successfully.',
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }

      this.$store.commit(`loadingStop`, 'admin-utilities-telemetry-resetid')
      this.loading = false
    }
  },
  apollo: {
    telemetry: {
      query: utilityTelemetryQuery,
      fetchPolicy: 'network-only',
      manual: true,
      result ({ data }) {
        this.telemetry = _.get(data, 'system.info.telemetry', false)
        this.clientId = _.get(data, 'system.info.telemetryClientId', 'N/A')
      },
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-utilities-telemetry-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
