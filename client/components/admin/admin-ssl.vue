<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-validation.svg" alt="SSL" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{ $t('admin:ssl.title') }}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft">{{ $t('admin:ssl.subtitle') }}</div>
          </div>
          <v-spacer></v-spacer>
          <v-btn class="animated fadeInDown" v-if="info.sslProvider === `letsencrypt` && info.httpsPort > 0" color="black" dark depressed @click="renewCertificate" large :loading="loadingRenew">
            <v-icon left>mdi-cached</v-icon><span>{{$t('admin:ssl.renewCertificate')}}</span>
          </v-btn>
        </div>
        <v-form class="pt-3">
          <v-layout row wrap>
            <v-flex lg6 xs12>
              <v-card class="animated fadeInUp">
                <v-subheader>{{ $t('admin:ssl.currentState') }}</v-subheader>
                <v-list two-line dense>
                  <v-list-item>
                    <v-list-item-avatar>
                      <v-icon class="indigo white--text">mdi-handshake</v-icon>
                    </v-list-item-avatar>
                    <v-list-item-content>
                      <v-list-item-title>{{ $t(`admin:ssl.provider`) }}</v-list-item-title>
                      <v-list-item-subtitle>{{ providerTitle }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                  <template v-if="info.sslProvider === `letsencrypt` && info.httpsPort > 0">
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="indigo white--text">mdi-application</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t(`admin:ssl.domain`) }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.sslDomain }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="indigo white--text">mdi-at</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t('admin:ssl.subscriberEmail') }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.sslSubscriberEmail }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="indigo white--text">mdi-calendar-remove-outline</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t('admin:ssl.expiration') }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.sslExpirationDate | moment('calendar') }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="indigo white--text">mdi-traffic-light</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t(`admin:ssl.status`) }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.sslStatus }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                  </template>
                </v-list>
              </v-card>
            </v-flex>
            <v-flex lg6 xs12>
              <v-card class="animated fadeInUp wait-p2s">
                <v-subheader>{{ $t('admin:ssl.ports') }}</v-subheader>
                <v-list two-line dense>
                  <v-list-item>
                    <v-list-item-avatar>
                      <v-icon class="blue white--text">mdi-lock-open-variant</v-icon>
                    </v-list-item-avatar>
                    <v-list-item-content>
                      <v-list-item-title>{{ $t(`admin:ssl.httpPort`) }}</v-list-item-title>
                      <v-list-item-subtitle>{{ info.httpPort }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                  <template v-if="info.httpsPort > 0">
                    <v-divider></v-divider>
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="green white--text">mdi-lock</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t(`admin:ssl.httpsPort`) }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.httpsPort }}</v-list-item-subtitle>
                      </v-list-item-content>
                    </v-list-item>
                    <v-divider></v-divider>
                    <v-list-item>
                      <v-list-item-avatar>
                        <v-icon class="indigo white--text">mdi-sign-direction</v-icon>
                      </v-list-item-avatar>
                      <v-list-item-content>
                        <v-list-item-title>{{ $t(`admin:ssl.httpPortRedirect`) }}</v-list-item-title>
                        <v-list-item-subtitle>{{ info.httpRedirection }}</v-list-item-subtitle>
                      </v-list-item-content>
                      <v-list-item-action>
                        <v-btn class="red--text" v-if="info.httpRedirection" depressed :color="$vuetify.theme.dark ? `red darken-4` : `red lighten-5`" :class="$vuetify.theme.dark ? `text--lighten-5` : `text--darken-2`" @click="toggleRedir" :loading="loadingRedir">
                          <v-icon left>mdi-power</v-icon><span>{{$t('admin:ssl.httpPortRedirectTurnOff')}}</span>
                        </v-btn>
                        <v-btn class="green--text" v-else depressed :color="$vuetify.theme.dark ? `green darken-4` : `green lighten-5`" :class="$vuetify.theme.dark ? `text--lighten-5` : `text--darken-2`" @click="toggleRedir" :loading="loadingRedir">
                          <v-icon left>mdi-power</v-icon><span>{{$t('admin:ssl.httpPortRedirectTurnOn')}}</span>
                        </v-btn>
                      </v-list-item-action>
                    </v-list-item>
                  </template>
                </v-list>
              </v-card>
            </v-flex>
          </v-layout>
        </v-form>
      </v-flex>
    </v-layout>
    <v-dialog v-model="loadingRenew" persistent max-width="450">
      <v-card color="black" dark>
        <v-card-text class="pa-10 text-center">
          <semipolar-spinner class="animated fadeIn" :animation-duration="1500" :size="65" color="#FFF" style="margin: 0 auto;"></semipolar-spinner>
          <div class="mt-5 body-1 white--text">{{$t('admin:ssl.renewCertificateLoadingTitle')}}</div>
          <div class="caption mt-4">{{$t('admin:ssl.renewCertificateLoadingSubtitle')}}</div>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import _ from 'lodash'
import gql from 'graphql-tag'

import { SemipolarSpinner } from 'epic-spinners'

export default {
  components: {
    SemipolarSpinner
  },
  data() {
    return {
      loadingRenew: false,
      loadingRedir: false,
      info: {
        sslDomain: '',
        sslProvider: '',
        sslSubscriberEmail: '',
        sslExpirationDate: false,
        sslStatus: '',
        httpPort: 0,
        httpRedirection: false,
        httpsPort: 0
      }
    }
  },
  computed: {
    providerTitle () {
      switch (this.info.sslProvider) {
        case 'custom':
          return this.$t('admin:ssl.providerCustomCertificate')
        case 'letsencrypt':
          return this.$t('admin:ssl.providerLetsEncrypt')
        default:
          return this.$t('admin:ssl.providerDisabled')
      }
    }
  },
  methods: {
    async toggleRedir () {
      this.loadingRedir = true
      try {
        this.info.httpRedirection = !this.info.httpRedirection
        await this.$apollo.mutate({
          mutation: gql`
            mutation ($enabled: Boolean!) {
              system {
                setHTTPSRedirection(enabled: $enabled) {
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
            enabled: _.get(this.info, 'httpRedirection', false)
          },
          watchLoading (isLoading) {
            this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-ssl-toggleRedirection')
          }
        })
        this.$store.commit('showNotification', {
          style: 'success',
          message: this.$t('admin:ssl.httpPortRedirectSaveSuccess'),
          icon: 'check'
        })
      } catch (err) {
        this.info.httpRedirection = !this.info.httpRedirection
        this.$store.commit('pushGraphError', err)
      }
      this.loadingRedir = false
    },
    async renewCertificate () {
      this.loadingRenew = true
      try {
        const respRaw = await this.$apollo.mutate({
          mutation: gql`
            mutation {
              system {
                renewHTTPSCertificate {
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
          watchLoading (isLoading) {
            this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-ssl-renew')
          }
        })
        const resp = _.get(respRaw, 'data.system.renewHTTPSCertificate.responseResult', {})
        if (resp.succeeded) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.$t('admin:ssl.renewCertificateSuccess'),
            icon: 'check'
          })
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
      }
      this.loadingRenew = false
    }
  },
  apollo: {
    info: {
      query: gql`
        {
          system {
            info {
              httpPort
              httpRedirection
              httpsPort
              sslDomain
              sslExpirationDate
              sslProvider
              sslStatus
              sslSubscriberEmail
            }
          }
        }
      `,
      fetchPolicy: 'network-only',
      update: (data) => _.cloneDeep(data.system.info),
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-ssl-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>

</style>
