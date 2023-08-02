<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-file.svg" alt="Page" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline blue--text text--darken-2 animated fadeInLeft">Pages</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">Manage pages</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p1s" icon color="grey" outlined @click="refresh">
            <v-icon class="grey--text">mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown mx-3" color="primary" outlined @click="recyclebin" disabled>
            <v-icon left>mdi-delete-outline</v-icon><span>Recycle Bin</span>
          </v-btn>
          <v-btn class="animated fadeInDown" color="primary" depressed large to="pages/visualize">
            <v-icon left>mdi-graph</v-icon><span>Visualize</span>
          </v-btn>
        </div>
        <v-card class="mt-3 animated fadeInUp">
          <div class="pa-2 d-flex align-center" :class="$vuetify.theme.dark ? `grey darken-3-d5` : `grey lighten-3`">
            <v-text-field solo flat v-model="search" prepend-inner-icon="mdi-file-search-outline" label="Search Pages..." hide-details dense style="max-width: 400px;"></v-text-field>
            <v-spacer></v-spacer>
            <v-select class="ml-2" solo flat hide-details dense label="Locale" :items="langs" v-model="selectedLang" style="max-width: 250px;"></v-select>
            <v-select class="ml-2" solo flat hide-details dense label="Publish State" :items="states" v-model="selectedState" style="max-width: 250px;"></v-select>
          </div>
          <v-divider></v-divider>
          <v-data-table :items="filteredPages" :headers="headers" :search="search" :page.sync="pagination" :items-per-page="15" :loading="loading" must-sort sort-by="updatedAt" sort-desc hide-default-footer @page-count="pageTotal = $event">
            <template slot="item" slot-scope="props">
              <tr class="is-clickable" :active="props.selected" @click="$router.push(`/pages/` + props.item.id)">
                <td class="text-xs-right">{{ props.item.id }}</td>
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
              <v-alert class="ma-3" icon="mdi-alert" :value="true" outlined>No pages to display.</v-alert>
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
import _ from 'lodash'
import pagesQuery from 'gql/admin/pages/pages-query-list.gql'

export default {
  data() {
    return {
      selectedPage: {},
      pagination: 1,
      pages: [],
      pageTotal: 0,
      headers: [
        { text: 'ID', value: 'id', width: 80, sortable: true },
        { text: 'Title', value: 'title' },
        { text: 'Path', value: 'path' },
        { text: 'Created', value: 'createdAt', width: 250 },
        { text: 'Last Updated', value: 'updatedAt', width: 250 }
      ],
      search: '',
      selectedLang: null,
      selectedState: null,
      states: [
        { text: 'All Publishing States', value: null },
        { text: 'Published', value: true },
        { text: 'Not Published', value: false }
      ],
      loading: false
    }
  },
  computed: {
    filteredPages () {
      return _.filter(this.pages, pg => {
        if (this.selectedLang !== null && this.selectedLang !== pg.locale) {
          return false
        }
        if (this.selectedState !== null && this.selectedState !== pg.isPublished) {
          return false
        }
        return true
      })
    },
    langs () {
      return _.concat({
        text: 'All Locales',
        value: null
      }, _.uniqBy(this.pages, 'locale').map(pg => ({
        text: pg.locale,
        value: pg.locale
      })))
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.pages.refetch()
      this.$store.commit('showNotification', {
        message: 'Page list has been refreshed.',
        style: 'success',
        icon: 'cached'
      })
    },
    newpage() {
      this.pageSelectorShown = true
    },
    recyclebin () { }
  },
  apollo: {
    pages: {
      query: pagesQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.pages.list,
      watchLoading (isLoading) {
        this.loading = isLoading
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-pages-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>
.admin-pages-path {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-family: 'Roboto Mono', monospace;
}
</style>
