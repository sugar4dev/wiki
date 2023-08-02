<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-customer.svg" alt="Users" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline blue--text text--darken-2 animated fadeInLeft">Users</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">Manage users</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p2s mr-3" outlined color="grey" icon @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown" color="primary" large depressed @click="createUser">
            <v-icon left>mdi-plus</v-icon><span>New User</span>
          </v-btn>
        </div>
        <v-card class="mt-3 animated fadeInUp">
          <div class="pa-2 d-flex align-center" :class="$vuetify.theme.dark ? `grey darken-3-d5` : `grey lighten-3`">
            <v-text-field solo flat v-model="search" prepend-inner-icon="mdi-account-search-outline" label="Search Users..." hide-details style="max-width: 400px;" dense></v-text-field>
            <v-spacer></v-spacer>
            <v-select solo flat hide-details label="Identity Provider" :items="strategies" v-model="filterStrategy" item-text="displayName" item-value="key" style="max-width: 300px;" dense></v-select>
          </div>
          <v-divider></v-divider>
          <v-data-table v-model="selected" :items="usersFiltered" :headers="headers" :search="search" :page.sync="pagination" :items-per-page="15" :loading="loading" @page-count="pageCount = $event" hide-default-footer>
            <template slot="item" slot-scope="props">
              <tr class="is-clickable" :active="props.selected" @click="$router.push('/users/' + props.item.id)">
                <td>{{ props.item.id }}</td>
                <td><strong>{{ props.item.name }}</strong></td>
                <td>{{ props.item.email }}</td>
                <td>{{ getStrategyName(props.item.providerKey) }}</td>
                <td>{{ props.item.createdAt | moment('from') }}</td>
                <td><span v-if="props.item.lastLoginAt">{{ props.item.lastLoginAt | moment('from') }}</span><em class="grey--text" v-else>Never</em></td>
                <td class="text-right">
                  <v-icon class="mr-3" v-if="props.item.isSystem">mdi-lock-outline</v-icon>
                  <status-indicator positive pulse v-if="props.item.isActive"></status-indicator>
                  <status-indicator negative pulse v-else></status-indicator>
                </td>
              </tr>
            </template>
            <template slot="no-data">
              <div class="pa-3">
                <v-alert class="text-left" icon="mdi-alert" outlined color="grey"><em class="body-2">No users to display!</em></v-alert>
              </div>
            </template>
          </v-data-table>
          <v-card-chin v-if="pageCount > 1">
            <v-spacer></v-spacer>
            <v-pagination v-model="pagination" :length="pageCount"></v-pagination>
            <v-spacer></v-spacer>
          </v-card-chin>
        </v-card>
      </v-flex>
    </v-layout>
    <user-create v-model="isCreateDialogShown" @refresh="refresh(false)"></user-create>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'

import { StatusIndicator } from 'vue-status-indicator'
import UserCreate from './admin-users-create.vue'

export default {
  components: {
    StatusIndicator,
    UserCreate
  },
  data() {
    return {
      selected: [],
      pagination: 1,
      pageCount: 0,
      users: [],
      headers: [
        { text: 'ID', value: 'id', width: 80, sortable: true },
        { text: 'Name', value: 'name', sortable: true },
        { text: 'Email', value: 'email', sortable: true },
        { text: 'Provider', value: 'provider', sortable: true },
        { text: 'Created', value: 'createdAt', sortable: true },
        { text: 'Last Login', value: 'lastLoginAt', sortable: true },
        { text: '', value: 'actions', sortable: false, width: 80 }
      ],
      strategies: [],
      filterStrategy: 'all',
      search: '',
      loading: false,
      isCreateDialogShown: false
    }
  },
  computed: {
    usersFiltered () {
      const all = this.filterStrategy === 'all' || this.filterStrategy === ''
      return _.filter(this.users, u => all || u.providerKey === this.filterStrategy)
    }
  },
  methods: {
    createUser() {
      this.isCreateDialogShown = true
    },
    async refresh(notify = true) {
      await this.$apollo.queries.users.refetch()
      if (notify) {
        this.$store.commit('showNotification', {
          message: 'Users list has been refreshed.',
          style: 'success',
          icon: 'cached'
        })
      }
    },
    getStrategyName(key) {
      return (_.find(this.strategies, ['key', key]) || {}).displayName || key
    }
  },
  apollo: {
    users: {
      query: gql`
        query {
          users {
            list {
              id
              name
              email
              providerKey
              isSystem
              isActive
              createdAt
              lastLoginAt
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => data.users.list,
      watchLoading (isLoading) {
        this.loading = isLoading
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-users-refresh')
      }
    },
    strategies: {
      query: gql`
        query {
          authentication {
            activeStrategies {
              key
              displayName
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => {
        return _.concat({
          key: 'all',
          displayName: 'All Providers'
        }, data.authentication.activeStrategies)
      },
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-users-strategies-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
