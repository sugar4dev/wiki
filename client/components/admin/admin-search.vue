<template>
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-search.svg" alt="Search Engine" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('admin:search.title')}}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">{{$t('admin:search.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="mr-3 animated fadeInDown wait-p3s" icon outlined color="grey" href="https://docs.requarks.io/search" target="_blank">
            <v-icon>mdi-help-circle</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown wait-p2s" icon outlined color="grey" @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="mx-3 animated fadeInDown wait-p1s" color="black" dark depressed @click="rebuild">
            <v-icon left>mdi-cached</v-icon><span>{{$t('admin:search.rebuildIndex')}}</span>
          </v-btn>
          <v-btn class="animated fadeInDown" color="success" @click="save" depressed large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
      </v-flex>
      <v-flex lg3 xs12>
        <v-card class="animated fadeInUp">
          <v-toolbar flat color="primary" dark dense>
            <div class="subtitle-1">{{$t('admin:search.searchEngine')}}</div>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <template v-for="(eng, idx) in engines">
              <v-list-item :key="eng.key" @click="selectedEngine = eng.key" :disabled="!eng.isAvailable">
                <v-list-item-avatar size="24">
                  <v-icon color="grey" v-if="!eng.isAvailable">mdi-minus-box-outline</v-icon>
                  <v-icon color="primary" v-else-if="eng.key === selectedEngine">mdi-checkbox-marked-circle-outline</v-icon>
                  <v-icon color="grey" v-else>mdi-checkbox-blank-circle-outline</v-icon>
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title class="body-2" :class="!eng.isAvailable ? `grey--text` : (selectedEngine === eng.key ? `primary--text` : ``)">{{ eng.title }}</v-list-item-title>
                  <v-list-item-subtitle>
                    <div class="caption" :class="!eng.isAvailable ? `grey--text text--lighten-1` : (selectedEngine === eng.key ? `blue--text ` : ``)">{{ eng.description }}</div>
                  </v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-avatar v-if="selectedEngine === eng.key" size="24">
                  <v-icon class="animated fadeInLeft" color="primary" large>mdi-chevron-right</v-icon>
                </v-list-item-avatar>
              </v-list-item>
              <v-divider v-if="idx < engines.length - 1"></v-divider>
            </template>
          </v-list>
        </v-card>
      </v-flex>
      <v-flex lg9 xs12>
        <v-card class="animated fadeInUp wait-p2s">
          <v-toolbar color="primary" dense flat dark>
            <div class="subtitle-1">{{engine.title}}</div>
          </v-toolbar>
          <v-card-info color="blue">
            <div>
              <div>{{engine.description}}</div><span class="caption"><a :href="engine.website">{{engine.website}}</a></span>
            </div>
            <v-spacer></v-spacer>
            <div class="admin-providerlogo"><img :src="engine.logo" :alt="engine.title"></div>
          </v-card-info>
          <v-card-text>
            <div class="overline mb-5">{{$t('admin:search.engineConfig')}}</div>
            <div class="body-2 ml-3" v-if="!engine.config || engine.config.length < 1"><em>{{$t('admin:search.engineNoConfig')}}</em></div>
            <template v-else v-for="cfg in engine.config">
              <v-select v-if="cfg.value.type === 'string' && cfg.value.enum" outlined :items="cfg.value.enum" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-select>
              <v-switch class="mb-3" v-else-if="cfg.value.type === 'boolean'" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" color="primary" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint inset></v-switch>
              <v-textarea v-else-if="cfg.value.type === 'string' && cfg.value.multiline" outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-textarea>
              <v-text-field v-else outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-text-field>
            </template>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'

import enginesQuery from 'gql/admin/search/search-query-engines.gql'
import enginesSaveMutation from 'gql/admin/search/search-mutation-save-engines.gql'
import enginesRebuildMutation from 'gql/admin/search/search-mutation-rebuild-index.gql'

export default {
  data() {
    return {
      engines: [],
      selectedEngine: '',
      engine: {}
    }
  },
  watch: {
    selectedEngine(newValue, oldValue) {
      this.engine = _.find(this.engines, ['key', newValue]) || {}
    },
    engines(newValue, oldValue) {
      this.selectedEngine = _.get(_.find(this.engines, 'isEnabled'), 'key', 'db')
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.engines.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:search.listRefreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    async save() {
      this.$store.commit(`loadingStart`, 'admin-search-saveengines')
      try {
        const resp = await this.$apollo.mutate({
          mutation: enginesSaveMutation,
          variables: {
            engines: this.engines.map(tgt => ({
              isEnabled: tgt.key === this.selectedEngine,
              key: tgt.key,
              config: tgt.config.map(cfg => ({...cfg, value: JSON.stringify({ v: cfg.value.value })}))
            }))
          }
        })
        if (_.get(resp, 'data.search.updateSearchEngines.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            message: this.$t('admin:search.configSaveSuccess'),
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(_.get(resp, 'data.search.updateSearchEngines.responseResult.message', this.$t('common:error.unexpected')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-search-saveengines')
    },
    async rebuild () {
      this.$store.commit(`loadingStart`, 'admin-search-rebuildindex')
      try {
        const resp = await this.$apollo.mutate({
          mutation: enginesRebuildMutation
        })
        if (_.get(resp, 'data.search.rebuildIndex.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            message: this.$t('admin:search.indexRebuildSuccess'),
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(_.get(resp, 'data.search.rebuildIndex.responseResult.message', this.$t('common:error.unexpected')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-search-rebuildindex')
    }
  },
  apollo: {
    engines: {
      query: enginesQuery,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.search.searchEngines).map(str => ({
        ...str,
        config: _.sortBy(str.config.map(cfg => ({
          ...cfg,
          value: JSON.parse(cfg.value)
        })), [t => t.value.order])
      })),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-search-refresh')
      }
    }
  }
}
</script>

<style lang='scss' scoped>

.enginelogo {
  width: 250px;
  height: 85px;
  float:right;
  display: flex;
  justify-content: flex-end;
  align-items: center;

  img {
    max-width: 100%;
    max-height: 50px;
  }
}

</style>
