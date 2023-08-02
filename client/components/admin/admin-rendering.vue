<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-process.svg" alt="Rendering" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{ $t('admin:rendering.title') }}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p4s">{{ $t('admin:rendering.subtitle') }}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p3s" icon outlined color="grey" href="https://docs.requarks.io/rendering" target="_blank">
            <v-icon>mdi-help-circle</v-icon>
          </v-btn>
          <v-btn class="mx-3 animated fadeInDown wait-p2s" icon outlined color="grey" @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown" color="success" @click="save" depressed large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
      </v-flex>
      <v-flex class="animated fadeInUp" lg3 xs12>
        <v-toolbar color="blue darken-2" dense flat dark>
          <div class="subtitle-1">Pipeline</div>
        </v-toolbar>
        <v-expansion-panels class="adm-rendering-pipeline" v-model="selectedCore" accordion mandatory>
          <v-expansion-panel v-for="core in renderers" :key="core.key">
            <v-expansion-panel-header hide-actions ripple>
              <v-toolbar color="blue" dense dark flat>
                <v-spacer></v-spacer>
                <div class="body-2">{{core.input}}</div>
                <v-icon class="mx-2">mdi-arrow-right-circle</v-icon>
                <div class="caption">{{core.output}}</div>
                <v-spacer></v-spacer>
              </v-toolbar>
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-list class="py-0" two-line dense>
                <template v-for="(rdr, n) in core.children">
                  <v-list-item :key="rdr.key" @click="selectRenderer(rdr.key)" :class="currentRenderer.key === rdr.key ? ($vuetify.theme.dark ? `grey darken-4-l4` : `blue lighten-5`) : ``">
                    <v-list-item-avatar size="24" tile>
                      <v-icon :color="currentRenderer.key === rdr.key ? 'primary' : 'grey'">{{rdr.icon}}</v-icon>
                    </v-list-item-avatar>
                    <v-list-item-content>
                      <v-list-item-title>{{rdr.title}}</v-list-item-title>
                      <v-list-item-subtitle>
                        <div class="caption">{{rdr.description}}</div>
                      </v-list-item-subtitle>
                    </v-list-item-content>
                    <v-list-item-avatar size="24">
                      <status-indicator v-if="rdr.isEnabled" positive pulse></status-indicator>
                      <status-indicator v-else negative pulse></status-indicator>
                    </v-list-item-avatar>
                  </v-list-item>
                  <v-divider class="my-0" v-if="n < core.children.length - 1"></v-divider>
                </template>
              </v-list>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-flex>
      <v-flex lg9 xs12>
        <v-card class="wiki-form animated fadeInUp">
          <v-toolbar color="indigo" dark flat dense>
            <v-icon class="mr-2">{{currentRenderer.icon}}</v-icon>
            <div class="subtitle-1">{{currentRenderer.title}}</div>
            <v-spacer></v-spacer>
            <v-switch dark color="white" label="Enabled" v-model="currentRenderer.isEnabled" hide-details inset></v-switch>
          </v-toolbar>
          <v-card-info color="blue">
            <div>
              <div>{{currentRenderer.description}}</div><span class="caption"><a href="https://docs.requarks.io/en/rendering" target="_blank">Documentation</a></span>
            </div>
          </v-card-info>
          <v-card-text class="pb-4 pl-4">
            <div class="overline mb-5">Rendering Module Configuration</div>
            <div class="body-2 ml-3" v-if="!currentRenderer.config || currentRenderer.config.length < 1"><em>This rendering module has no configuration options you can modify.</em></div>
            <template v-else v-for="(cfg, idx) in currentRenderer.config">
              <v-select v-if="cfg.value.type === 'string' && cfg.value.enum" outlined :items="cfg.value.enum" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" color="indigo"></v-select>
              <v-switch v-else-if="cfg.value.type === 'boolean'" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" color="indigo" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint inset></v-switch>
              <v-text-field v-else outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" color="indigo"></v-text-field>
              <v-divider class="my-5" v-if="idx < currentRenderer.config.length - 1"></v-divider>
            </template>
          </v-card-text>
          <v-card-chin>
            <v-spacer></v-spacer>
            <div class="caption pr-3 grey--text">Module: {{ currentRenderer.key }}</div>
          </v-card-chin>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'
import { DepGraph } from 'dependency-graph'

import { StatusIndicator } from 'vue-status-indicator'

import renderersQuery from 'gql/admin/rendering/rendering-query-renderers.gql'
import renderersSaveMutation from 'gql/admin/rendering/rendering-mutation-save-renderers.gql'

export default {
  components: {
    StatusIndicator
  },
  data() {
    return {
      selectedCore: -1,
      renderers: [],
      currentRenderer: {}
    }
  },
  watch: {
    renderers(newValue, oldValue) {
      _.delay(() => {
        this.selectedCore = _.findIndex(newValue, ['key', 'markdownCore'])
        this.selectRenderer('markdownCore')
      }, 500)
    }
  },
  methods: {
    selectRenderer (key) {
      this.renderers.map(rdr => {
        if (_.some(rdr.children, ['key', key])) {
          this.currentRenderer = _.find(rdr.children, ['key', key])
        }
      })
    },
    async refresh () {
      await this.$apollo.queries.renderers.refetch()
      this.$store.commit('showNotification', {
        message: 'Rendering active configuration has been reloaded.',
        style: 'success',
        icon: 'cached'
      })
    },
    async save () {
      this.$store.commit(`loadingStart`, 'admin-rendering-saverenderers')
      await this.$apollo.mutate({
        mutation: renderersSaveMutation,
        variables: {
          renderers: _.reduce(this.renderers, (result, core) => {
            result = _.concat(result, core.children.map(rd => ({
              key: rd.key,
              isEnabled: rd.isEnabled,
              config: rd.config.map(cfg => ({ key: cfg.key, value: JSON.stringify({ v: cfg.value.value }) }))
            })))
            return result
          }, [])
        }
      })
      this.$store.commit('showNotification', {
        message: 'Rendering configuration saved successfully.',
        style: 'success',
        icon: 'check'
      })
      this.$store.commit(`loadingStop`, 'admin-rendering-saverenderers')
    }
  },
  apollo: {
    renderers: {
      query: renderersQuery,
      fetchPolicy: 'network-only',
      update: (data) => {
        let renderers = _.cloneDeep(data.rendering.renderers).map(str => ({
          ...str,
          config: _.sortBy(str.config.map(cfg => ({
            ...cfg,
            value: JSON.parse(cfg.value)
          })), [t => t.value.order])
        }))
        // Build tree
        const graph = new DepGraph({ circular: true })
        const rawCores = _.filter(renderers, ['dependsOn', null]).map(core => {
          core.children = _.concat([_.cloneDeep(core)], _.filter(renderers, ['dependsOn', core.key]))
          return core
        })
        // Build dependency graph
        rawCores.map(core => { graph.addNode(core.key) })
        rawCores.map(core => {
          rawCores.map(coreTarget => {
            if (core.key !== coreTarget.key) {
              if (core.output === coreTarget.input) {
                graph.addDependency(core.key, coreTarget.key)
              }
            }
          })
        })
        // Reorder cores in reverse dependency order
        let orderedCores = []
        _.reverse(graph.overallOrder()).map(coreKey => {
          orderedCores.push(_.find(rawCores, ['key', coreKey]))
        })
        return orderedCores
      },
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-rendering-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>
.adm-rendering-pipeline {
  .v-expansion-panel--active .v-expansion-panel-header {
    min-height: 0;
  }

  .v-expansion-panel-header {
    padding: 0;
    margin-top: 1px;
  }

  .v-expansion-panel-content__wrap {
    padding: 0;
  }
}
</style>
