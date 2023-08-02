<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="profile-header"><img class="animated fadeInUp" src="/_assets/svg/icon-file.svg" alt="Users" style="width: 80px;">
          <div class="profile-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('profile:pages.title')}}</div>
            <div class="subheading grey--text animated fadeInLeft">{{$t('profile:pages.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p1s" color="grey" outlined @click="refresh" large>
            <v-icon class="grey--text">mdi-refresh</v-icon>
          </v-btn>
        </div>
      </v-flex>
      <v-flex xs12>
        <v-card class="animated fadeInUp">
          <v-data-table :items="pages" :headers="headers" :page.sync="pagination" :items-per-page="15" :loading="loading" must-sort sort-by="updatedAt" sort-desc hide-default-footer>
            <template slot="item" slot-scope="props">
              <tr class="is-clickable" :active="props.selected" @click="goToPage(props.item.id)">
                <td>
                  <div class="body-2"><strong>{{ props.item.title }}</strong></div>
                  <div class="caption">{{ props.item.description }}</div>
                </td>
                <td class="admin-pages-path">
                  <v-chip label small :color="$vuetify.theme.dark ? `grey darken-4` : `grey lighten-4`">{{ props.item.locale }}</v-chip><span class="ml-2 grey--text" :class="$vuetify.theme.dark ? `text--lighten-1` : `text--darken-2`">/ {{ props.item.path }}</span>
                </td>
                <td>{{ props.item.createdAt | moment('calendar') }}</td>
                <td>{{ props.item.updatedAt | moment('calendar') }}</td>
              </tr>
            </template>
            <template slot="no-data">
              <v-alert class="ma-3" icon="mdi-alert" :value="true" outlined color="grey"><em class="caption">{{$t('profile:pages.emptyList')}}</em></v-alert>
            </template>
          </v-data-table>
          <div class="text-center py-2 animated fadeInDown" v-if="this.pageTotal > 1">
            <v-pagination v-model="pagination" :length="pageTotal"></v-pagination>
          </div>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import gql from 'graphql-tag'

export default {
  data() {
    return {
      selectedPage: {},
      pagination: 1,
      pages: [],
      loading: false
    }
  },
  computed: {
    headers () {
      return [
        { text: this.$t('profile:pages.headerTitle'), value: 'title' },
        { text: this.$t('profile:pages.headerPath'), value: 'path' },
        { text: this.$t('profile:pages.headerCreatedAt'), value: 'createdAt', width: 250 },
        { text: this.$t('profile:pages.headerUpdatedAt'), value: 'updatedAt', width: 250 }
      ]
    },
    pageTotal () {
      return Math.ceil(this.pages.length / 15)
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.pages.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('profile:pages.refreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    goToPage(id) {
      window.location.assign(`/i/` + id)
    }
  },
  apollo: {
    pages: {
      query: gql`
        query($creatorId: Int, $authorId: Int) {
          pages {
            list(creatorId: $creatorId, authorId: $authorId) {
              id
              locale
              path
              title
              description
              contentType
              isPublished
              isPrivate
              privateNS
              createdAt
              updatedAt
            }
          }
        }
      `,
      variables () {
        return {
          creatorId: this.$store.get('user/id'),
          authorId: this.$store.get('user/id')
        }
      },
      fetchPolicy: 'network-only',
      update: (data) => data.pages.list,
      watchLoading (isLoading) {
        this.loading = isLoading
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'profile-pages-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
