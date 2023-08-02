<template>  
  <v-app class="tags" :dark="$vuetify.theme.dark">
    <nav-header></nav-header>
    <v-navigation-drawer class="pb-0 elevation-1" app fixed clipped :right="$vuetify.rtl" permanent width="300">
      <vue-scroll :ops="scrollStyle">
        <v-list dense nav>
          <v-list-item href="/">
            <v-list-item-icon>
              <v-icon>mdi-home</v-icon>
            </v-list-item-icon>
            <v-list-item-title>{{$t('common:header.home')}}</v-list-item-title>
          </v-list-item>
          <template v-for="(tags, groupName) in tagsGrouped">
            <v-divider class="my-2"></v-divider>
            <v-subheader class="pl-4" :key="`tagGroup-` + groupName">{{groupName}}</v-subheader>
            <v-list-item v-for="tag of tags" @click="toggleTag(tag.tag)" :key="`tag-` + tag.tag">
              <v-list-item-icon>
                <v-icon v-if="isSelected(tag.tag)" color="primary">mdi-checkbox-intermediate</v-icon>
                <v-icon v-else>mdi-checkbox-blank-outline</v-icon>
              </v-list-item-icon>
              <v-list-item-title>{{tag.title}}</v-list-item-title>
            </v-list-item>
          </template>
        </v-list>
      </vue-scroll>
    </v-navigation-drawer>
    <v-content class="grey" :class="$vuetify.theme.dark ? `darken-4-d5` : `lighten-3`">
      <v-toolbar color="primary" dark flat height="58">
        <template v-if="selection.length > 0">
          <div class="overline mr-3 animated fadeInLeft">{{$t('tags:currentSelection')}}</div>
          <v-chip class="mr-3 primary--text" v-for="tag of tagsSelected" :key="`tagSelected-` + tag.tag" color="white" close @click:close="toggleTag(tag.tag)">{{tag.title}}</v-chip>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeIn" small outlined color="blue lighten-4" rounded @click="selection = []">
            <v-icon left>mdi-close</v-icon><span>{{$t('tags:clearSelection')}}</span>
          </v-btn>
        </template>
        <template v-else>
          <v-icon class="mr-3 animated fadeInRight">mdi-arrow-left</v-icon>
          <div class="overline animated fadeInRight">{{$t('tags:selectOneMoreTags')}}</div>
        </template>
      </v-toolbar>
      <v-toolbar :color="$vuetify.theme.dark ? `grey darken-4-l5` : `grey lighten-4`" flat height="58">
        <v-text-field class="tags-search" v-model="innerSearch" :label="$t(`tags:searchWithinResultsPlaceholder`)" solo hide-details flat rounded single-line height="40" prepend-icon="mdi-text-box-search-outline" append-icon="mdi-arrow-right" clearable></v-text-field>
        <template v-if="locales.length > 1">
          <v-divider class="mx-3" vertical></v-divider>
          <div class="overline">{{$t('tags:locale')}}</div>
          <v-select class="ml-2" :items="locales" v-model="locale" :background-color="$vuetify.theme.dark ? `grey darken-3` : `white`" hide-details :label="$t(`tags:locale`)" item-text="name" item-value="code" rounded single-line dense height="40" style="max-width: 170px;"></v-select>
        </template>
        <v-divider class="mx-3" vertical></v-divider>
        <div class="overline">{{$t('tags:orderBy')}}</div>
        <v-select class="ml-2" :items="orderByItems" v-model="orderBy" :background-color="$vuetify.theme.dark ? `grey darken-3` : `white`" hide-details :label="$t(`tags:orderBy`)" rounded single-line dense height="40" style="max-width: 250px;"></v-select>
        <v-btn-toggle class="ml-2" v-model="orderByDirection" rounded mandatory>
          <v-btn text height="40">
            <v-icon size="20">mdi-chevron-double-up</v-icon>
          </v-btn>
          <v-btn text height="40">
            <v-icon size="20">mdi-chevron-double-down</v-icon>
          </v-btn>
        </v-btn-toggle>
      </v-toolbar>
      <v-divider></v-divider>
      <div class="text-center pt-10" v-if="selection.length < 1"><img src="/_assets/svg/icon-price-tag.svg">
        <div class="subtitle-2 grey--text">{{$t('tags:selectOneMoreTagsHint')}}</div>
      </div>
      <div class="px-5 py-2" v-else>
        <v-data-iterator :items="pages" :items-per-page="4" :search="innerSearch" :loading="isLoading" :options.sync="pagination" hide-default-footer ref="dude">
          <template v-slot:loading>
            <div class="text-center pt-10">
              <v-progress-circular indeterminate color="primary" size="96" width="2"></v-progress-circular>
              <div class="subtitle-2 grey--text mt-5">{{$t('tags:retrievingResultsLoading')}}</div>
            </div>
          </template>
          <template v-slot:no-data>
            <div class="text-center pt-10"><img src="/_assets/svg/icon-info.svg">
              <div class="subtitle-2 grey--text">{{$t('tags:noResults')}}</div>
            </div>
          </template>
          <template v-slot:no-results>
            <div class="text-center pt-10"><img src="/_assets/svg/icon-info.svg">
              <div class="subtitle-2 grey--text">{{$t('tags:noResultsWithFilter')}}</div>
            </div>
          </template>
          <template v-slot:default="props">
            <v-row align="stretch">
              <v-col v-for="item of props.items" :key="`page-` + item.id" cols="12" lg="6">
                <v-card class="radius-7" @click="goTo(item)" style="height:100%;" :class="$vuetify.theme.dark ? `grey darken-4` : ``">
                  <v-card-text>
                    <div class="d-flex flex-row align-center">
                      <div class="body-1"><strong class="primary--text">{{item.title}}</strong></div>
                      <v-spacer></v-spacer>
                      <i18next class="caption" tag="div" path="tags:pageLastUpdated"><span place="date">{{item.updatedAt | moment('from')}}</span></i18next>
                    </div>
                    <div class="body-2 grey--text">{{item.description || '---'}}</div>
                    <v-divider class="my-2"></v-divider>
                    <div class="d-flex flex-row align-center">
                      <v-chip class="overline" small label :color="$vuetify.theme.dark ? `grey darken-3-l5` : `grey lighten-4`">{{item.locale}}</v-chip>
                      <div class="caption ml-1">/ {{item.path}}</div>
                    </div>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>
          </template>
        </v-data-iterator>
        <div class="text-center py-2 animated fadeInDown" v-if="this.pageTotal > 1">
          <v-pagination v-model="pagination.page" :length="pageTotal"></v-pagination>
        </div>
      </div>
    </v-content>
    <nav-footer></nav-footer>
    <notify></notify>
    <search-results></search-results>
  </v-app>
</template>

<script>
import VueRouter from 'vue-router'
import _ from 'lodash'

import tagsQuery from 'gql/common/common-pages-query-tags.gql'
import pagesQuery from 'gql/common/common-pages-query-list.gql'

/* global siteLangs */

const router = new VueRouter({
  mode: 'history',
  base: '/t'
})

export default {
  i18nOptions: { namespaces: 'tags' },
  data() {
    return {
      tags: [],
      selection: [],
      innerSearch: '',
      locale: 'any',
      locales: [],
      orderBy: 'title',
      orderByDirection: 0,
      pagination: {
        page: 1,
        itemsPerPage: 12,
        mustSort: true,
        sortBy: ['title'],
        sortDesc: [false]
      },
      pages: [],
      isLoading: true,
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
    tagsGrouped () {
      return _.groupBy(this.tags, t => t.title.charAt(0).toUpperCase())
    },
    tagsSelected () {
      return _.filter(this.tags, t => _.includes(this.selection, t.tag))
    },
    pageTotal () {
      return Math.ceil(this.pages.length / this.pagination.itemsPerPage)
    },
    orderByItems () {
      return [
        { text: this.$t('tags:orderByField.creationDate'), value: 'createdAt' },
        { text: this.$t('tags:orderByField.ID'), value: 'id' },
        { text: this.$t('tags:orderByField.lastModified'), value: 'updatedAt' },
        { text: this.$t('tags:orderByField.path'), value: 'path' },
        { text: this.$t('tags:orderByField.title'), value: 'title' }
      ]
    }
  },
  watch: {
    locale (newValue, oldValue) {
      this.rebuildURL()
    },
    orderBy (newValue, oldValue) {
      this.rebuildURL()
      this.pagination.sortBy = [newValue]
    },
    orderByDirection (newValue, oldValue) {
      this.rebuildURL()
      this.pagination.sortDesc = [newValue === 1]
    }
  },
  router,
  created () {
    this.$store.commit('page/SET_MODE', 'tags')
    this.selection = _.compact(decodeURI(this.$route.path).split('/'))
  },
  mounted () {
    this.locales = _.concat(
      [{name: this.$t('tags:localeAny'), code: 'any'}],
      (siteLangs.length > 0 ? siteLangs : [])
    )
    if (this.$route.query.lang) {
      this.locale = this.$route.query.lang
    }
    if (this.$route.query.sort) {
      this.orderBy = this.$route.query.sort.toLowerCase()
      switch (this.orderBy) {
        case 'updatedat':
          this.orderBy = 'updatedAt'
          break
      }
      this.pagination.sortBy = [this.orderBy]
    }
    if (this.$route.query.dir) {
      this.orderByDirection = this.$route.query.dir === 'asc' ? 0 : 1
      this.pagination.sortDesc = [this.orderByDirection === 1]
    }
  },
  methods: {
    toggleTag (tag) {
      if (_.includes(this.selection, tag)) {
        this.selection = _.without(this.selection, tag)
      } else {
        this.selection.push(tag)
      }
      this.rebuildURL()
    },
    isSelected (tag) {
      return _.includes(this.selection, tag)
    },
    rebuildURL () {
      let urlObj = {
        path: '/' + this.selection.join('/')
      }
      if (this.locale !== `any`) {
        _.set(urlObj, 'query.lang', this.locale)
      }
      if (this.orderBy !== `TITLE`) {
        _.set(urlObj, 'query.sort', this.orderBy.toLowerCase())
      }
      if (this.orderByDirection !== 0) {
        _.set(urlObj, 'query.dir', this.orderByDirection === 0 ? `asc` : `desc`)
      }
      this.$router.push(urlObj)
    },
    goTo (page) {
      window.location.assign(`/${page.locale}/${page.path}`)
    }
  },
  apollo: {
    tags: {
      query: tagsQuery,
      fetchPolicy: 'cache-and-network',
      update: (data) => _.cloneDeep(data.pages.tags),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'tags-refresh')
      }
    },
    pages: {
      query: pagesQuery,
      fetchPolicy: 'cache-and-network',
      update: (data) => _.cloneDeep(data.pages.list),
      watchLoading (isLoading) {
        this.isLoading = isLoading
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'pages-refresh')
      },
      variables () {
        return {
          locale: this.locale === 'any' ? null : this.locale,
          tags: this.selection
        }
      },
      skip () {
        return this.selection.length < 1
      }
    }
  }
}
</script>

<style lang='scss'>
.tags-search {
  .v-input__control {
    min-height: initial !important;
  }
  .v-input__prepend-outer {
    margin-top: 8px !important;
  }
}
</style>
