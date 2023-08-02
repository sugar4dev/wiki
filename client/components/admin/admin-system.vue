<template>  
  <v-container class="admin-system" fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img class="animated fadeInUp" src="/_assets/svg/icon-tune.svg" alt="System Info" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text animated fadeInLeft">{{ $t('admin:system.title') }}</div>
            <div class="subtitle-1 grey--text animated fadeInLeft wait-p2s">{{ $t('admin:system.subtitle') }}</div>
          </div>
        </div>
        <v-layout class="mt-3" row wrap>
          <v-flex lg6 xs12>
            <v-card class="animated fadeInUp">
              <v-btn class="animated fadeInLeft wait-p2s btn-animate-rotate" fab absolute :right="!$vuetify.rtl" :left="$vuetify.rtl" top small light @click="refresh">
                <v-icon color="grey">mdi-refresh</v-icon>
              </v-btn>
              <v-subheader>Wiki.js</v-subheader>
              <v-list two-line dense>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue white--text">mdi-application-export</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.currentVersion') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.currentVersion }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue white--text">mdi-inbox-arrow-up</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.latestVersion') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.latestVersion }}</v-list-item-subtitle>
                  </v-list-item-content>
                  <v-list-item-action>
                    <v-list-item-action-text>{{ $t('admin:system.published') }} {{ info.latestVersionReleaseDate | moment('from') }}</v-list-item-action-text>
                  </v-list-item-action>
                </v-list-item>
              </v-list>
              <v-card-actions v-if="info.upgradeCapable && !isLatestVersion && info.platform === `docker`" :class="$vuetify.theme.dark ? `grey darken-3-d5` : `indigo lighten-5`">
                <div class="caption indigo--text pl-3" :class="$vuetify.theme.dark ? `text--lighten-4` : ``">Wiki.js can perform the upgrade to the latest version for you.</div>
                <v-spacer></v-spacer>
                <v-btn class="px-3" color="indigo" dark @click="performUpgrade">
                  <v-icon left>mdi-upload</v-icon><span>Perform Upgrade</span>
                </v-btn>
              </v-card-actions>
            </v-card>
            <v-card class="mt-4 animated fadeInUp wait-p2s">
              <v-subheader>{{ $t('admin:system.hostInfo') }}</v-subheader>
              <v-list two-line dense>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-avatar class="blue-grey" size="40">
                      <v-icon color="white">{{platformLogo}}</v-icon>
                    </v-avatar>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.os') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ (info.platform === 'docker') ? 'Docker Container (Linux)' : info.operatingSystem }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue-grey white--text">mdi-desktop-classic</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.hostname') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.hostname }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue-grey white--text">mdi-cpu-64-bit</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.cpuCores') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.cpuCores }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue-grey white--text">mdi-memory</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.totalRAM') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.ramTotal }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue-grey white--text">mdi-iframe-outline</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.workingDirectory') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.workingDirectory }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-icon class="blue-grey white--text">mdi-card-bulleted-settings-outline</v-icon>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ $t('admin:system.configFile') }}</v-list-item-title>
                    <v-list-item-subtitle>{{ info.configFile }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </v-flex>
          <v-flex lg6 xs12>
            <v-card class="pb-3 animated fadeInUp wait-p4s">
              <v-subheader>Node.js</v-subheader>
              <v-list dense>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-avatar class="light-green" size="40">
                      <v-icon color="white">mdi-nodejs</v-icon>
                    </v-avatar>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title>{{ info.nodeVersion }}</v-list-item-title>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
              <v-divider class="mt-3"></v-divider>
              <v-subheader>{{ info.dbType }}</v-subheader>
              <v-list dense>
                <v-list-item>
                  <v-list-item-avatar>
                    <v-avatar class="indigo darken-1" size="40">
                      <v-icon color="white">mdi-database</v-icon>
                    </v-avatar>
                  </v-list-item-avatar>
                  <v-list-item-content>
                    <v-list-item-title v-html="dbVersion"></v-list-item-title>
                    <v-list-item-subtitle>{{ info.dbHost }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-alert class="mt-3 mx-4" :value="isDbLimited" color="deep-orange darken-2" icon="mdi-alert" dark>{{ $t('admin:system.dbPartialSupport') }}</v-alert>
              </v-list>
            </v-card>
          </v-flex>
        </v-layout>
      </v-flex>
    </v-layout>
    <v-dialog v-model="isUpgrading" persistent width="450">
      <v-card class="blue darken-5" dark>
        <v-card-text class="text-center pa-10">
          <self-building-square-spinner :animation-duration="4000" :size="40" color="#FFF" style="margin: 0 auto;"></self-building-square-spinner>
          <div class="body-2 mt-5 blue--text text--lighten-4">Your Wiki.js container is being upgraded...</div>
          <div class="caption blue--text text--lighten-2">Please wait</div>
          <v-progress-linear class="mt-5" color="blue lighten-2" :value="upgradeProgress" :buffer-value="upgradeProgress" rounded :stream="isUpgradingStarted" query :indeterminate="!isUpgradingStarted"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import _ from 'lodash'

import { SelfBuildingSquareSpinner } from 'epic-spinners'

import systemInfoQuery from 'gql/admin/system/system-query-info.gql'
import performUpgradeMutation from 'gql/admin/system/system-mutation-upgrade.gql'

export default {
  components: {
    SelfBuildingSquareSpinner
  },
  data () {
    return {
      isUpgrading: false,
      isUpgradingStarted: false,
      upgradeProgress: 0,
      info: {}
    }
  },
  computed: {
    dbVersion () {
      return _.get(this.info, 'dbVersion', '').replace(/(?:\r\n|\r|\n)/g, '<br />')
    },
    platformLogo () {
      switch (this.info.platform) {
        case 'docker':
          return 'mdi-docker'
        case 'darwin':
          return 'mdi-apple'
        case 'linux':
          if (this.info.operatingSystem.indexOf('Ubuntu')) {
            return 'mdi-ubuntu'
          } else {
            return 'mdi-linux'
          }
        case 'win32':
          return 'mdi-microsoft-windows'
        default:
          return ''
      }
    },
    isDbLimited () {
      return this.info.dbType === 'MySQL' && this.dbVersion.indexOf('5.') === 0
    },
    isLatestVersion () {
      return this.info.currentVersion === this.info.latestVersion
    }
  },
  methods: {
    async refresh () {
      await this.$apollo.queries.info.refetch()
      this.$store.commit('showNotification', {
        message: this.$t('admin:system.refreshSuccess'),
        style: 'success',
        icon: 'cached'
      })
    },
    async performUpgrade () {
      this.isUpgrading = true
      this.isUpgradingStarted = false
      this.upgradeProgress = 0
      this.$store.commit(`loadingStart`, 'admin-system-upgrade')
      try {
        const respRaw = await this.$apollo.mutate({
          mutation: performUpgradeMutation
        })
        const resp = _.get(respRaw, 'data.system.performUpgrade.responseResult', {})
        if (resp.succeeded) {
          this.isUpgradingStarted = true
          let progressInterval = setInterval(() => {
            this.upgradeProgress += 0.83
          }, 500)
          _.delay(() => {
            clearInterval(progressInterval)
            window.location.reload(true)
          }, 60000)
        } else {
          throw new Error(resp.message)
        }
      } catch (err) {
        this.$store.commit('pushGraphError', err)
        this.$store.commit(`loadingStop`, 'admin-system-upgrade')
        this.isUpgrading = false
      }
    }
  },
  apollo: {
    info: {
      query: systemInfoQuery,
      fetchPolicy: 'network-only',
      update: (data) => data.system.info,
      watchLoading (isLoading) {
        this.$store.commit(`loading${isLoading ? 'Start' : 'Stop'}`, 'admin-system-refresh')
      }
    }
  }
}
</script>

<style lang='scss'>
.admin-system {
  .v-list-item-title, .v-list-item__subtitle {
    user-select: text;
  }
}
</style>
