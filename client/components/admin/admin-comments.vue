<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-chat-bubble.svg" alt="Comments" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{$t('admin:comments.title')}}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">{{$t('admin:comments.subtitle')}}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown wait-p3s" icon outlined color="grey" href="https://docs.requarks.io/comments" target="_blank">
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
      <v-flex lg3 xs12>
        <v-card class="animated fadeInUp">
          <v-toolbar flat color="primary" dark dense>
            <div class="subtitle-1">{{$t('admin:comments.provider')}}</div>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <template v-for="(provider, idx) in providers">
              <v-list-item :key="provider.key" @click="selectedProvider = provider.key" :disabled="!provider.isAvailable">
                <v-list-item-avatar size="24">
                  <v-icon color="grey" v-if="!provider.isAvailable">mdi-minus-box-outline</v-icon>
                  <v-icon color="primary" v-else-if="provider.key === selectedProvider">mdi-checkbox-marked-circle-outline</v-icon>
                  <v-icon color="grey" v-else>mdi-checkbox-blank-circle-outline</v-icon>
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title class="body-2" :class="!provider.isAvailable ? `grey--text` : (selectedProvider === provider.key ? `primary--text` : ``)">{{ provider.title }}</v-list-item-title>
                  <v-list-item-subtitle>
                    <div class="caption" :class="!provider.isAvailable ? `grey--text text--lighten-1` : (selectedProvider === provider.key ? `blue--text ` : ``)">{{ provider.description }}</div>
                  </v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-avatar v-if="selectedProvider === provider.key" size="24">
                  <v-icon class="animated fadeInLeft" color="primary" large>mdi-chevron-right</v-icon>
                </v-list-item-avatar>
              </v-list-item>
              <v-divider v-if="idx < providers.length - 1"></v-divider>
            </template>
          </v-list>
        </v-card>
      </v-flex>
      <v-flex lg9 xs12>
        <v-card class="animated fadeInUp wait-p2s">
          <v-toolbar color="primary" dense flat dark>
            <div class="subtitle-1">{{provider.title}}</div>
          </v-toolbar>
          <v-card-info color="blue">
            <div>
              <div>{{provider.description}}</div><span class="caption"><a :href="provider.website">{{provider.website}}</a></span>
            </div>
            <v-spacer></v-spacer>
            <div class="admin-providerlogo"><img :src="provider.logo" :alt="provider.title"></div>
          </v-card-info>
          <v-card-text>
            <div class="overline my-5">{{$t('admin:comments.providerConfig')}}</div>
            <div class="body-2 ml-3" v-if="!provider.config || provider.config.length < 1"><em>{{$t('admin:comments.providerNoConfig')}}</em></div>
            <template v-else v-for="cfg in provider.config">
              <v-select class="mb-3" v-if="cfg.value.type === 'string' && cfg.value.enum" outlined :items="cfg.value.enum" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" :style="cfg.value.maxWidth > 0 ? `max-width:` + cfg.value.maxWidth + `px;` : ``"></v-select>
              <v-switch class="mb-6" v-else-if="cfg.value.type === 'boolean'" :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" color="primary" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint inset></v-switch>
              <v-textarea class="mb-3" v-else-if="cfg.value.type === 'string' && cfg.value.multiline" outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''"></v-textarea>
              <v-text-field class="mb-3" v-else outlined :key="cfg.key" :label="cfg.value.title" v-model="cfg.value.value" prepend-icon="mdi-cog-box" :hint="cfg.value.hint ? cfg.value.hint : ''" persistent-hint :class="cfg.value.hint ? 'mb-2' : ''" :style="cfg.value.maxWidth > 0 ? `max-width:` + cfg.value.maxWidth + `px;` : ``"></v-text-field>
            </template>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'

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
      this.selectedProvider = _.get(_.find(this.providers, 'isEnabled'), 'key', 'db')
    }
  },
  methods: {
    async refresh() {
      await this.$apollo.queries.providers.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:comments.listRefreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    async save() {
      this.$store.commit(`loadingStart`, 'admin-comments-saveproviders')
      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation($providers: [CommentProviderInput]!) {
              comments {
                updateProviders(providers: $providers) {
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
            providers: this.providers.map(tgt => ({
              isEnabled: tgt.key === this.selectedProvider,
              key: tgt.key,
              config: tgt.config.map(cfg => ({...cfg, value: JSON.stringify({ v: cfg.value.value })}))
            }))
          }
        })
        if (_.get(resp, 'data.comments.updateProviders.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            message: this.$t('admin:comments.configSaveSuccess'),
            style: 'success',
            icon: 'check'
          })
        } else {
          throw new Error(_.get(resp, 'data.comments.updateProviders.responseResult.message', this.$t('common:error.unexpected')))
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.$store.commit(`loadingStop`, 'admin-comments-saveproviders')
    }
  },
  apollo: {
    providers: {
      query: gql`
        query {
          comments {
            providers {
              isEnabled
              key
              title
              description
              logo
              website
              isAvailable
              config {
                key
                value
              }
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.comments.providers).map(str => ({
        ...str,
        config: _.sortBy(str.config.map(cfg => ({
          ...cfg,
          value: JSON.parse(cfg.value)
        })), [t => t.value.order])
      })),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-comments-refresh')
      }
    }
  }
}
</script>
