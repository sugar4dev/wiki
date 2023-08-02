<template>  
  <v-container fluid grid-list-lg>
    <v-layout row wrap>
      <v-flex xs12>
        <div class="admin-header"><img src="/_assets/svg/icon-maintenance.svg" alt="Utilities" style="width: 80px;">
          <div class="admin-header-title">
            <div class="headline primary--text">{{$t('admin:utilities.title')}}</div>
            <div class="subtitle-1 grey--text">{{$t('admin:utilities.subtitle')}}</div>
          </div>
        </div>
      </v-flex>
      <v-flex lg3 xs12>
        <v-card class="animated fadeInUp">
          <v-toolbar flat color="primary" dark dense>
            <div class="subtitle-1">{{$t('admin:utilities.tools')}}</div>
          </v-toolbar>
          <v-list class="py-0" two-line dense>
            <template v-for="(tool, idx) in tools">
              <v-list-item :key="tool.key" @click="selectedTool = tool.key" :disabled="!tool.isAvailable">
                <v-list-item-avatar>
                  <v-icon :color="!tool.isAvailable ? `grey lighten-1` : (selectedTool === tool.key ? `blue ` : `grey darken-1`)">{{ tool.icon }}</v-icon>
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title class="body-2" :class="!tool.isAvailable ? `grey--text` : (selectedTool === tool.key ? `primary--text` : ``)">{{ $t('admin:utilities.' + tool.i18nKey + 'Title') }}</v-list-item-title>
                  <v-list-item-subtitle>
                    <div class="caption" :class="!tool.isAvailable ? `grey--text text--lighten-1` : (selectedTool === tool.key ? `blue--text ` : ``)">{{ $t('admin:utilities.' + tool.i18nKey + 'Subtitle') }}</div>
                  </v-list-item-subtitle>
                </v-list-item-content>
                <v-list-item-avatar v-if="selectedTool === tool.key">
                  <v-icon class="animated fadeInLeft" color="primary" large>mdi-chevron-right</v-icon>
                </v-list-item-avatar>
              </v-list-item>
              <v-divider v-if="idx < tools.length - 1"></v-divider>
            </template>
          </v-list>
        </v-card>
      </v-flex>
      <v-flex class="animated fadeInUp wait-p2s" xs12 lg9>
        <transition name="admin-router">
          <component :is="selectedTool"></component>
        </transition>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>

export default {
  components: {
    UtilityAuth: () => import(/* webpackChunkName: "admin" */ './admin-utilities-auth.vue'),
    UtilityContent: () => import(/* webpackChunkName: "admin" */ './admin-utilities-content.vue'),
    UtilityCache: () => import(/* webpackChunkName: "admin" */ './admin-utilities-cache.vue'),
    UtilityExport: () => import(/* webpackChunkName: "admin" */ './admin-utilities-export.vue'),
    UtilityImportv1: () => import(/* webpackChunkName: "admin" */ './admin-utilities-importv1.vue'),
    UtilityTelemetry: () => import(/* webpackChunkName: "admin" */ './admin-utilities-telemetry.vue')
  },
  data() {
    return {
      selectedTool: 'UtilityAuth',
      tools: [
        {
          key: 'UtilityAuth',
          icon: 'mdi-lock-open-outline',
          i18nKey: 'auth',
          isAvailable: true
        },
        {
          key: 'UtilityContent',
          icon: 'mdi-content-duplicate',
          i18nKey: 'content',
          isAvailable: true
        },
        {
          key: 'UtilityExport',
          icon: 'mdi-database-export',
          i18nKey: 'export',
          isAvailable: true
        },
        {
          key: 'UtilityCache',
          icon: 'mdi-database-refresh',
          i18nKey: 'cache',
          isAvailable: true
        },
        // {
        //   key: 'UtilityGraphEndpoint',
        //   icon: 'mdi-graphql',
        //   i18nKey: 'graphEndpoint',
        //   isAvailable: false
        // },
        {
          key: 'UtilityImportv1',
          icon: 'mdi-database-import',
          i18nKey: 'importv1',
          isAvailable: true
        },
        {
          key: 'UtilityTelemetry',
          icon: 'mdi-math-compass',
          i18nKey: 'telemetry',
          isAvailable: true
        }
      ]
    }
  }
}
</script>

<style lang='scss'>

</style>
