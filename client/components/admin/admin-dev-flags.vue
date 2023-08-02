<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img src="/_assets/svg/icon-console.svg" alt="Developer Tools" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text">Developer Tools</div>
            <div class="subtitle-1 grey--text">Flags</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn color="success" depressed @click="save" large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
        <v-card class="mt-3" :class="$vuetify.theme.dark ? `grey darken-3-d5` : `white grey--text text--darken-3`">
          <v-alert color="red" :value="true" icon="mdi-alert" dark prominent><span>Do NOT enable these flags unless you know what you're doing!</span>
            <div class="caption">Doing so may result in data loss or broken installation!</div>
          </v-alert>
          <v-card-text>
            <v-switch class="mt-3" color="primary" hint="Log detailed debug info on LDAP/AD login attempts." persistent-hint label="LDAP Debug" v-model="flags.ldapdebug" inset></v-switch>
            <v-divider class="mt-3"></v-divider>
            <v-switch class="mt-3" color="red" hint="Log all queries made to the database to console." persistent-hint label="SQL Query Logging" v-model="flags.sqllog" inset></v-switch>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'

import flagsQuery from 'gql/admin/dev/dev-query-flags.gql'
import flagsMutation from 'gql/admin/dev/dev-mutation-save-flags.gql'

export default {
  data() {
    return {
      flags: {
        sqllog: false
      }
    }
  },
  methods: {
    async save() {
      try {
        await this.$apollo.mutate({
          mutation: flagsMutation,
          variables: {
            flags: _.transform(this.flags, (result, value, key) => {
              result.push({ key, value })
            }, [])
          },
          watchLoading (isLoading) {
            this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-dev-flags-update')
          }
        })
        this.$store.commit('showNotification', {
          style: 'success',
          message: 'Flags applied successfully.',
          icon: 'check'
        })
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
    }
  },
  apollo: {
    flags: {
      query: flagsQuery,
      fetchPolicy: 'network-only',
      update: (data) => _.transform(data.system.flags, (result, row) => {
        _.set(result, row.key, row.value)
      }, {}),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-dev-flags-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
