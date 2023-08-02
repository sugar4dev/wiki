<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-line-chart.svg" alt="Analytics" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{ $t('admin:analytics.title') }}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p4s">{{ $t('admin:analytics.subtitle') }}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p2s mr-3" icon outlined color="grey" @click="refresh">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
          <v-btn class="animated fadeInDown" color="success" @click="save" depressed large>
            <v-icon left>mdi-check</v-icon><span>{{$t('common:actions.apply')}}</span>
          </v-btn>
        </div>
      </v-flex>
      <v-flex lg3 xs12>
        <v-card class="animated fadeInUp">
          <v-toolbar flat color="primary" dark dense>
            <div class="subtitle-1">{{$t('admin:analytics.providers')}}</div>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <template v-for="(str, idx) in providers">
              <v-list-item :key="str.key" @click="selectedProvider = str.key" :disabled="!str.isAvailable">
                <v-list-item-avatar size="24">
                  <v-icon color="grey" v-if="!str.isAvailable">mdi-minus-box-outline</v-icon>
                  <v-icon color="primary" v-else-if="str.isEnabled" v-ripple @click="str.isEnabled = false">mdi-checkbox-marked-outline</v-icon>
                  <v-icon color="grey" v-else v-ripple @click="str.isEnabled = true">mdi-checkbox-blank-outline</v-icon>
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title class="body-2" :class="!str.isAvailable ? `grey--text` : (selectedProvider === str.key ? `primary--text` : ``)">{{ str.title }}</v-list-item-title>
                  <v-list-item-subtitle>
                    <div class="caption" :class="!str.isAvailable ? `grey--text text--lighten-1` : (selectedProvider === str.key ? `blue--text ` : ``)">{{ str.description }}</div>
                  </v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-avatar v-if="selectedProvider === str.key" size="24">
                  <v-icon class="animated fadeInLeft" color="primary" large>mdi-chevron-right</v-icon>
                </v-list-item-avatar>
              </v-list-item>
              <v-divider v-if="idx < providers.length - 1"></v-divider>
            </template>
          </v-list>
        </v-card>
      </v-flex>
      <v-flex xs12 lg9>
        <v-card class="animated fadeInUp wait-p2s">
          <v-toolbar color="primary" dense flat dark>
            <div class="subtitle-1">{{provider.title}}</div>
            <v-spacer></v-spacer>
            <v-switch dark color="blue lighten-5" label="Active" v-model="provider.isEnabled" hide-details inset></v-switch>
          </v-toolbar>
          <v-card-info color="blue">
            <div>
              <div>{{provider.description}}</div><span class="caption"><a :href="provider.website">{{provider.website}}</a></span>
            </div>
            <v-spacer></v-spacer>
            <div class="admin-providerlogo"><img :src="provider.logo" :alt="provider.title"></div>
          </v-card-info>
          <v-card-text>
            <v-form>
              <div class="overline pb-5">{{$t('admin:analytics.providerConfiguration')}}</div>
              <div class="body-1 ml-3" v-if="!provider.config || provider.config.length < 1"><em>{{$t('admin:analytics.providerNoConfiguration')}}</em></div>
              <template v-else v-for="cfg in provider.config">
                <v-select v-if="cfg.value.type === 'string' && cfg.value.enum" outlined :items="cfg.value.enum" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-select>
                <v-switch class="mb-3" v-else-if="cfg.value.type === 'boolean'" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" color="primary" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint inset></v-switch>
                <v-textarea v-else-if="cfg.value.type === 'string' && cfg.value.multiline" outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-textarea>
                <v-text-field v-else outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-text-field>
              </template>
            </v-form>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'

import providersQuery from 'gql/admin/analytics/analytics-query-providers.gql'
import providersSaveMutation from 'gql/admin/analytics/analytics-mutation-save-providers.gql'

export default {
  data() {
    return {
      providers: [],
      selectedProvider: '',
      provider: {}
    }
  },
  watch: {
    selectedProvider(newValue, oldValue) {
      this.provider = _.find(this.providers, ['key', newValue]) || {}
    },
    providers(newValue, oldValue) {
      this.selectedProvider = 'google'
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.providers.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:analytics.refreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    async save() {
      this.$store.commit(`loadingStart`, 'admin-analytics-saveproviders')
      try {
        await this.$apollo.mutate({
          mutation: providersSaveMutation,
          variables: {
            providers: this.providers.map(str => _.pick(str, [
              'isEnabled',
              'key',
              'config'
            ])).map(str => ({...str, config: str.config.map(cfg => ({...cfg, value: JSON.stringify({ v: cfg.value.value })}))}))
          }
        })
        this.$store.commit('showNotification', {
          message: this.$t('admin:analytics.saveSuccess'),
          style: 'success',
          icon: 'check'
        })
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-analytics-saveproviders')
    }
  },
  apollo: {
    providers: {
      query: providersQuery,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.analytics.providers).map(str => ({
        ...str,
        config: _.sortBy(str.config.map(cfg => ({
          ...cfg,
          value: JSON.parse(cfg.value)
        })), [t => t.value.order])
      })),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-analytics-refresh')
      }
    }
  }
}
</script>
