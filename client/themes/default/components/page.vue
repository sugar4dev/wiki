<template>  
  <v-app v-scroll="upBtnScroll" :dark="$vuetify.theme.dark" :class="$vuetify.rtl ? `is-rtl` : `is-ltr`">
    <nav-header v-if="!printView"></nav-header>
    <v-navigation-drawer v-if="navMode !== `NONE` && !printView" :class="$vuetify.theme.dark ? `grey darken-4-d4` : `primary`" dark app clipped mobile-breakpoint="600" :temporary="$vuetify.breakpoint.smAndDown" v-model="navShown" :right="$vuetify.rtl">
      <vue-scroll :ops="scrollStyle">
        <nav-sidebar :color="$vuetify.theme.dark ? `grey darken-4-d4` : `primary`" :items="sidebarDecoded" :nav-mode="navMode"></nav-sidebar>
      </vue-scroll>
    </v-navigation-drawer>
    <v-fab-transition v-if="navMode !== `NONE`">
      <v-btn fab color="primary" fixed bottom :right="$vuetify.rtl" :left="!$vuetify.rtl" small @click="navShown = !navShown" v-if="$vuetify.breakpoint.mdAndDown" v-show="!navShown">
        <v-icon>mdi-menu</v-icon>
      </v-btn>
    </v-fab-transition>
    <v-main ref="content">
      <template v-if="path !== `home`">
        <v-toolbar :color="$vuetify.theme.dark ? `grey darken-4-d3` : `grey lighten-3`" flat dense v-if="$vuetify.breakpoint.smAndUp">
          <v-breadcrumbs class="breadcrumbs-nav pl-0" :items="breadcrumbs" divider="/">
            <template slot="item" slot-scope="props">
              <v-icon v-if="props.item.path === '/'" small @click="goHome">mdi-home</v-icon>
              <v-btn class="ma-0" v-else :href="props.item.path" small text>{{props.item.name}}</v-btn>
            </template>
          </v-breadcrumbs>
          <template v-if="!isPublished">
            <v-spacer></v-spacer>
            <div class="caption red--text">{{$t('common:page.unpublished')}}</div>
            <status-indicator class="ml-3" negative pulse></status-indicator>
          </template>
        </v-toolbar>
        <v-divider></v-divider>
      </template>
      <v-container class="grey pa-0" fluid :class="$vuetify.theme.dark ? `darken-4-l3` : `lighten-4`">
        <v-row class="page-header-section" no-gutters align-content="center" style="height: 90px;">
          <v-col class="page-col-content is-page-header" :offset-xl="tocPosition === `left` ? 2 : 0" :offset-lg="tocPosition === `left` ? 3 : 0" :xl="tocPosition === `right` ? 10 : false" :lg="tocPosition === `right` ? 9 : false" style="margin-top: auto; margin-bottom: auto;" :class="$vuetify.rtl ? `pr-4` : `pl-4`">
            <div class="page-header-headings">
              <div class="headline grey--text" :class="$vuetify.theme.dark ? `text--lighten-2` : `text--darken-3`">{{title}}</div>
              <div class="caption grey--text text--darken-1">{{description}}</div>
            </div>
            <div class="page-edit-shortcuts" v-if="editShortcutsObj.editMenuBar" :class="tocPosition === `right` ? `is-right` : ``">
              <v-btn v-if="editShortcutsObj.editMenuBtn" @click="pageEdit" depressed small>
                <v-icon class="mr-2" small>mdi-pencil</v-icon><span class="text-none">{{$t(`common:actions.edit`)}}</span>
              </v-btn>
              <v-btn v-if="editShortcutsObj.editMenuExternalBtn" :href="editMenuExternalUrl" target="_blank" depressed small>
                <v-icon class="mr-2" small>{{ editShortcutsObj.editMenuExternalIcon }}</v-icon><span class="text-none">{{$t(`common:page.editExternal`, { name: editShortcutsObj.editMenuExternalName })}}</span>
              </v-btn>
            </div>
          </v-col>
        </v-row>
      </v-container>
      <v-divider></v-divider>
      <v-container class="pl-5 pt-4" fluid grid-list-xl>
        <v-layout row>
          <v-flex class="page-col-sd" v-if="tocPosition !== `off` && $vuetify.breakpoint.lgAndUp" :order-xs1="tocPosition !== `right`" :order-xs2="tocPosition === `right`" lg3 xl2>
            <v-card class="page-toc-card mb-5" v-if="tocDecoded.length">
              <div class="overline pa-5 pb-0" :class="$vuetify.theme.dark ? `blue--text text--lighten-2` : `primary--text`">{{$t('common:page.toc')}}</div>
              <v-list class="pb-3" dense nav :class="$vuetify.theme.dark ? `darken-3-d3` : ``">
                <template v-for="(tocItem, tocIdx) in tocDecoded">
                  <v-list-item @click="$vuetify.goTo(tocItem.anchor, scrollOpts)">
                    <v-icon color="grey" small>{{ $vuetify.rtl ? `mdi-chevron-left` : `mdi-chevron-right` }}</v-icon>
                    <v-list-item-title class="px-3">{{tocItem.title}}</v-list-item-title>
                  </v-list-item>
                  <template v-for="tocSubItem in tocItem.children">
                    <v-list-item @click="$vuetify.goTo(tocSubItem.anchor, scrollOpts)">
                      <v-icon class="px-3" color="grey lighten-1" small>{{ $vuetify.rtl ? `mdi-chevron-left` : `mdi-chevron-right` }}</v-icon>
                      <v-list-item-title class="px-3 caption grey--text" :class="$vuetify.theme.dark ? `text--lighten-1` : `text--darken-1`">{{tocSubItem.title}}</v-list-item-title>
                    </v-list-item>
                  </template>
                </template>
              </v-list>
            </v-card>
            <v-card class="page-tags-card mb-5" v-if="tags.length > 0">
              <div class="pa-5">
                <div class="overline teal--text pb-2" :class="$vuetify.theme.dark ? `text--lighten-3` : ``">{{$t('common:page.tags')}}</div>
                <v-chip class="mr-1 mb-1" label :color="$vuetify.theme.dark ? `teal darken-1` : `teal lighten-5`" v-for="(tag, idx) in tags" :href="`/t/` + tag.tag" :key="`tag-` + tag.tag">
                  <v-icon :color="$vuetify.theme.dark ? `teal lighten-3` : `teal`" left small>mdi-tag</v-icon><span :class="$vuetify.theme.dark ? `teal--text text--lighten-5` : `teal--text text--darken-2`">{{tag.title}}</span>
                </v-chip>
                <v-chip class="mr-1 mb-1" label :color="$vuetify.theme.dark ? `teal darken-1` : `teal lighten-5`" :href="`/t/` + tags.map(t => t.tag).join(`/`)" :aria-label="$t(`common:page.tagsMatching`)">
                  <v-icon :color="$vuetify.theme.dark ? `teal lighten-3` : `teal`" size="20">mdi-tag-multiple</v-icon>
                </v-chip>
              </div>
            </v-card>
            <v-card class="page-comments-card mb-5" v-if="commentsEnabled && commentsPerms.read">
              <div class="pa-5">
                <div class="overline pb-2 blue-grey--text d-flex align-center" :class="$vuetify.theme.dark ? `text--lighten-3` : `text--darken-2`"><span>{{$t('common:comments.sdTitle')}}</span></div>
                <div class="d-flex">
                  <v-btn class="text-none" @click="goToComments()" :color="$vuetify.theme.dark ? `blue-grey` : `blue-grey darken-2`" outlined style="flex: 1 1 100%;" small><span class="blue-grey--text" :class="$vuetify.theme.dark ? `text--lighten-1` : `text--darken-2`">{{$t('common:comments.viewDiscussion')}}</span></v-btn>
                  <v-tooltip right v-if="commentsPerms.write">
                    <template v-slot:activator="{ on }">
                      <v-btn class="ml-2" @click="goToComments(true)" v-on="on" outlined small :color="$vuetify.theme.dark ? `blue-grey` : `blue-grey darken-2`" :aria-label="$t(`common:comments.newComment`)">
                        <v-icon :color="$vuetify.theme.dark ? `blue-grey lighten-1` : `blue-grey darken-2`" dense>mdi-comment-plus</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:comments.newComment')}}</span>
                  </v-tooltip>
                </div>
              </div>
            </v-card>
            <v-card class="page-author-card mb-5">
              <div class="pa-5">
                <div class="overline indigo--text d-flex" :class="$vuetify.theme.dark ? `text--lighten-3` : ``"><span>{{$t('common:page.lastEditedBy')}}</span>
                  <v-spacer></v-spacer>
                  <v-tooltip right v-if="isAuthenticated">
                    <template v-slot:activator="{ on }">
                      <v-btn class="btn-animate-edit" icon :href="'/h/' + locale + '/' + path" v-on="on" x-small v-if="hasReadHistoryPermission" :aria-label="$t(`common:header.history`)">
                        <v-icon color="indigo" dense>mdi-history</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.history')}}</span>
                  </v-tooltip>
                </div>
                <div class="page-author-card-name body-2 grey--text" :class="$vuetify.theme.dark ? `` : `text--darken-3`">{{ authorName }}</div>
                <div class="page-author-card-date caption grey--text text--darken-1">{{ updatedAt | moment('calendar') }}</div>
              </div>
            </v-card>
            <v-card class="page-shortcuts-card" flat>
              <v-toolbar :color="$vuetify.theme.dark ? `grey darken-4-d3` : `grey lighten-3`" flat dense>
                <v-spacer></v-spacer>
                <v-menu offset-y bottom min-width="300">
                  <template v-slot:activator="{ on: menu }">
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on: tooltip }">
                        <v-btn icon tile v-on="{ ...menu, ...tooltip }" :aria-label="$t(`common:page.share`)">
                          <v-icon color="grey">mdi-share-variant</v-icon>
                        </v-btn>
                      </template><span>{{$t('common:page.share')}}</span>
                    </v-tooltip>
                  </template>
                  <social-sharing :url="pageUrl" :title="title" :description="description"></social-sharing>
                </v-menu>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-btn icon tile v-on="on" @click="print" :aria-label="$t(`common:page.printFormat`)">
                      <v-icon :color="printView ? `primary` : `grey`">mdi-printer</v-icon>
                    </v-btn>
                  </template><span>{{$t('common:page.printFormat')}}</span>
                </v-tooltip>
                <v-spacer></v-spacer>
              </v-toolbar>
            </v-card>
          </v-flex>
          <v-flex class="page-col-content" xs12 :lg9="tocPosition !== `off`" :xl10="tocPosition !== `off`" :order-xs1="tocPosition === `right`" :order-xs2="tocPosition !== `right`">
            <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasAnyPagePermissions && editShortcutsObj.editFab">
              <template v-slot:activator="{ on: onEditActivator }">
                <v-speed-dial v-model="pageEditFab" direction="top" open-on-hover transition="scale-transition" bottom :right="!$vuetify.rtl" :left="$vuetify.rtl" fixed dark>
                  <template v-slot:activator>
                    <v-btn class="btn-animate-edit" fab color="primary" v-model="pageEditFab" @click="pageEdit" v-on="onEditActivator" :disabled="!hasWritePagesPermission" :aria-label="$t(`common:page.editPage`)">
                      <v-icon>mdi-pencil</v-icon>
                    </v-btn>
                  </template>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasReadHistoryPermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab small color="white" light v-on="on" @click="pageHistory">
                        <v-icon size="20">mdi-history</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.history')}}</span>
                  </v-tooltip>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasReadSourcePermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab small color="white" light v-on="on" @click="pageSource">
                        <v-icon size="20">mdi-code-tags</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.viewSource')}}</span>
                  </v-tooltip>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasWritePagesPermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab small color="white" light v-on="on" @click="pageConvert">
                        <v-icon size="20">mdi-lightning-bolt</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.convert')}}</span>
                  </v-tooltip>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasWritePagesPermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab small color="white" light v-on="on" @click="pageDuplicate">
                        <v-icon size="20">mdi-content-duplicate</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.duplicate')}}</span>
                  </v-tooltip>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasManagePagesPermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab small color="white" light v-on="on" @click="pageMove">
                        <v-icon size="20">mdi-content-save-move-outline</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.move')}}</span>
                  </v-tooltip>
                  <v-tooltip :right="$vuetify.rtl" :left="!$vuetify.rtl" v-if="hasDeletePagesPermission">
                    <template v-slot:activator="{ on }">
                      <v-btn fab dark small color="red" v-on="on" @click="pageDelete">
                        <v-icon size="20">mdi-trash-can-outline</v-icon>
                      </v-btn>
                    </template><span>{{$t('common:header.delete')}}</span>
                  </v-tooltip>
                </v-speed-dial>
              </template><span>{{$t('common:page.editPage')}}</span>
            </v-tooltip>
            <v-alert class="mb-5" v-if="!isPublished" color="red" outlined icon="mdi-minus-circle" dense>
              <div class="caption">{{$t('common:page.unpublishedWarning')}}</div>
            </v-alert>
            <div class="contents" ref="container">
              <slot name="contents"></slot>
            </div>
            <div class="comments-container" id="discussion" v-if="commentsEnabled && commentsPerms.read && !printView">
              <div class="comments-header">
                <v-icon class="mr-2" dark>mdi-comment-text-outline</v-icon><span>{{$t('common:comments.title')}}</span>
              </div>
              <div class="comments-main">
                <slot name="comments"></slot>
              </div>
            </div>
          </v-flex>
        </v-layout>
      </v-container>
    </v-main>
    <nav-footer></nav-footer>
    <notify></notify>
    <search-results></search-results>
    <v-fab-transition>
      <v-btn v-if="upBtnShown" fab fixed bottom :right="$vuetify.rtl" :left="!$vuetify.rtl" small :depressed="this.$vuetify.breakpoint.mdAndUp" @click="$vuetify.goTo(0, scrollOpts)" color="primary" dark :style="upBtnPosition" :aria-label="$t(`common:actions.returnToTop`)">
        <v-icon>mdi-arrow-up</v-icon>
      </v-btn>
    </v-fab-transition>
  </v-app>
</template>

<script>
import { StatusIndicator } from 'vue-status-indicator'
import Tabset from './tabset.vue'
import NavSidebar from './nav-sidebar.vue'
import Prism from 'prismjs'
import mermaid from 'mermaid'
import { get, sync } from 'vuex-pathify'
import _ from 'lodash'
import ClipboardJS from 'clipboard'
import Vue from 'vue'

Vue.component('Tabset', Tabset)

Prism.plugins.autoloader.languages_path = '/_assets/js/prism/'
Prism.plugins.NormalizeWhitespace.setDefaults({
  'remove-trailing': true,
  'remove-indent': true,
  'left-trim': true,
  'right-trim': true,
  'remove-initial-line-feed': true,
  'tabs-to-spaces': 2
})
Prism.plugins.toolbar.registerButton('copy-to-clipboard', (env) => {
  let linkCopy = document.createElement('button')
  linkCopy.textContent = 'Copy'

  const clip = new ClipboardJS(linkCopy, {
    text: () => { return env.code }
  })

  clip.on('success', () => {
    linkCopy.textContent = 'Copied!'
    resetClipboardText()
  })
  clip.on('error', () => {
    linkCopy.textContent = 'Press Ctrl+C to copy'
    resetClipboardText()
  })

  return linkCopy

  function resetClipboardText() {
    setTimeout(() => {
      linkCopy.textContent = 'Copy'
    }, 5000)
  }
})

export default {
  components: {
    NavSidebar,
    StatusIndicator
  },
  props: {
    pageId: {
      type: Number,
      default: 0
    },
    locale: {
      type: String,
      default: 'en'
    },
    path: {
      type: String,
      default: 'home'
    },
    title: {
      type: String,
      default: 'Untitled Page'
    },
    description: {
      type: String,
      default: ''
    },
    createdAt: {
      type: String,
      default: ''
    },
    updatedAt: {
      type: String,
      default: ''
    },
    tags: {
      type: Array,
      default: () => ([])
    },
    authorName: {
      type: String,
      default: 'Unknown'
    },
    authorId: {
      type: Number,
      default: 0
    },
    editor: {
      type: String,
      default: ''
    },
    isPublished: {
      type: Boolean,
      default: false
    },
    toc: {
      type: String,
      default: ''
    },
    sidebar: {
      type: String,
      default: ''
    },
    navMode: {
      type: String,
      default: 'MIXED'
    },
    commentsEnabled: {
      type: Boolean,
      default: false
    },
    effectivePermissions: {
      type: String,
      default: ''
    },
    commentsExternal: {
      type: Boolean,
      default: false
    },
    editShortcuts: {
      type: String,
      default: ''
    },
    filename: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      navShown: false,
      navExpanded: false,
      upBtnShown: false,
      pageEditFab: false,
      scrollOpts: {
        duration: 1500,
        offset: 0,
        easing: 'easeInOutCubic'
      },
      scrollStyle: {
        vuescroll: {},
        scrollPanel: {
          initialScrollX: 0.01, // fix scrollbar not disappearing on load
          scrollingX: false,
          speed: 50
        },
        rail: {
          gutterOfEnds: '2px'
        },
        bar: {
          onlyShowBarOnScroll: false,
          background: '#42A5F5',
          hoverStyle: {
            background: '#64B5F6'
          }
        }
      },
      winWidth: 0
    }
  },
  computed: {
    isAuthenticated: get('user/authenticated'),
    commentsCount: get('page/commentsCount'),
    commentsPerms: get('page/effectivePermissions@comments'),
    editShortcutsObj: get('page/editShortcuts'),
    rating: {
      get () {
        return 3.5
      },
      set (val) {

      }
    },
    breadcrumbs() {
      return [{ path: '/', name: 'Home' }].concat(_.reduce(this.path.split('/'), (result, value, key) => {
        result.push({
          path: _.get(_.last(result), 'path', `/${this.locale}`) + `/${value}`,
          name: value
        })
        return result
      }, []))
    },
    pageUrl () { return window.location.href },
    upBtnPosition () {
      if (this.$vuetify.breakpoint.mdAndUp) {
        return this.$vuetify.rtl ? `right: 235px;` : `left: 235px;`
      } else {
        return this.$vuetify.rtl ? `right: 65px;` : `left: 65px;`
      }
    },
    sidebarDecoded () {
      return JSON.parse(Buffer.from(this.sidebar, 'base64').toString())
    },
    tocDecoded () {
      return JSON.parse(Buffer.from(this.toc, 'base64').toString())
    },
    tocPosition: get('site/tocPosition'),
    hasAdminPermission: get('page/effectivePermissions@system.manage'),
    hasWritePagesPermission: get('page/effectivePermissions@pages.write'),
    hasManagePagesPermission: get('page/effectivePermissions@pages.manage'),
    hasDeletePagesPermission: get('page/effectivePermissions@pages.delete'),
    hasReadSourcePermission: get('page/effectivePermissions@source.read'),
    hasReadHistoryPermission: get('page/effectivePermissions@history.read'),
    hasAnyPagePermissions () {
      return this.hasAdminPermission || this.hasWritePagesPermission || this.hasManagePagesPermission ||
        this.hasDeletePagesPermission || this.hasReadSourcePermission || this.hasReadHistoryPermission
    },
    printView: sync('site/printView'),
    editMenuExternalUrl () {
      if (this.editShortcutsObj.editMenuBar && this.editShortcutsObj.editMenuExternalBtn) {
        return this.editShortcutsObj.editMenuExternalUrl.replace('{filename}', this.filename)
      } else {
        return ''
      }
    }
  },
  created() {
    this.$store.set('page/authorId', this.authorId)
    this.$store.set('page/authorName', this.authorName)
    this.$store.set('page/createdAt', this.createdAt)
    this.$store.set('page/description', this.description)
    this.$store.set('page/isPublished', this.isPublished)
    this.$store.set('page/id', this.pageId)
    this.$store.set('page/locale', this.locale)
    this.$store.set('page/path', this.path)
    this.$store.set('page/tags', this.tags)
    this.$store.set('page/title', this.title)
    this.$store.set('page/editor', this.editor)
    this.$store.set('page/updatedAt', this.updatedAt)
    if (this.effectivePermissions) {
      this.$store.set('page/effectivePermissions', JSON.parse(Buffer.from(this.effectivePermissions, 'base64').toString()))
    }
    if (this.editShortcuts) {
      this.$store.set('page/editShortcuts', JSON.parse(Buffer.from(this.editShortcuts, 'base64').toString()))
    }

    this.$store.set('page/mode', 'view')
  },
  mounted () {
    if (this.$vuetify.theme.dark) {
      this.scrollStyle.bar.background = '#424242'
    }

    // -> Check side navigation visibility
    this.handleSideNavVisibility()
    window.addEventListener('resize', _.debounce(() => {
      this.handleSideNavVisibility()
    }, 500))

    // -> Highlight Code Blocks
    Prism.highlightAllUnder(this.$refs.container)

    // -> Render Mermaid diagrams
    mermaid.mermaidAPI.initialize({
      startOnLoad: true,
      theme: this.$vuetify.theme.dark ? `dark` : `default`
    })

    // -> Handle anchor scrolling
    if (window.location.hash && window.location.hash.length > 1) {
      if (document.readyState === 'complete') {
        this.$nextTick(() => {
          this.$vuetify.goTo(decodeURIComponent(window.location.hash), this.scrollOpts)
        })
      } else {
        window.addEventListener('load', () => {
          this.$vuetify.goTo(decodeURIComponent(window.location.hash), this.scrollOpts)
        })
      }
    }

    // -> Handle anchor links within the page contents
    this.$nextTick(() => {
      this.$refs.container.querySelectorAll(`a[href^="#"], a[href^="${window.location.href.replace(window.location.hash, '')}#"]`).forEach(el => {
        el.onclick = ev => {
          ev.preventDefault()
          ev.stopPropagation()
          this.$vuetify.goTo(decodeURIComponent(ev.currentTarget.hash), this.scrollOpts)
        }
      })

      window.boot.notify('page-ready')
    })
  },
  methods: {
    goHome () {
      window.location.assign('/')
    },
    toggleNavigation () {
      this.navOpen = !this.navOpen
    },
    upBtnScroll () {
      const scrollOffset = window.pageYOffset || document.documentElement.scrollTop
      this.upBtnShown = scrollOffset > window.innerHeight * 0.33
    },
    print () {
      if (this.printView) {
        this.printView = false
      } else {
        this.printView = true
        this.$nextTick(() => {
          window.print()
        })
      }
    },
    pageEdit () {
      this.$root.$emit('pageEdit')
    },
    pageHistory () {
      this.$root.$emit('pageHistory')
    },
    pageSource () {
      this.$root.$emit('pageSource')
    },
    pageConvert () {
      this.$root.$emit('pageConvert')
    },
    pageDuplicate () {
      this.$root.$emit('pageDuplicate')
    },
    pageMove () {
      this.$root.$emit('pageMove')
    },
    pageDelete () {
      this.$root.$emit('pageDelete')
    },
    handleSideNavVisibility () {
      if (window.innerWidth === this.winWidth) { return }
      this.winWidth = window.innerWidth
      if (this.$vuetify.breakpoint.mdAndUp) {
        this.navShown = true
      } else {
        this.navShown = false
      }
    },
    goToComments (focusNewComment = false) {
      this.$vuetify.goTo('#discussion', this.scrollOpts)
      if (focusNewComment) {
        document.querySelector('#discussion-new').focus()
      }
    }
  }
}
</script>

<style lang="scss">

.breadcrumbs-nav {
  .v-btn {
    min-width: 0;
    &__content {
      text-transform: none;
    }
  }
  .v-breadcrumbs__divider:nth-child(2n) {
    padding: 0 6px;
  }
  .v-breadcrumbs__divider:nth-child(2) {
    padding: 0 6px 0 12px;
  }
}

.page-col-sd {
  margin-top: -90px;
  align-self: flex-start;
  position: sticky;
  top: 64px;
  max-height: calc(100vh - 64px);
  overflow-y: auto;
  -ms-overflow-style: none;
}

.page-col-sd::-webkit-scrollbar {
  display: none;
}

.page-header-section {
  position: relative;

  > .is-page-header {
    position: relative;
  }

  .page-header-headings {
    min-height: 52px;
    display: flex;
    justify-content: center;
    flex-direction: column;
  }

  .page-edit-shortcuts {
    position: absolute;
    bottom: -33px;
    right: 10px;

    .v-btn {
      border-right: 1px solid #DDD !important;
      border-bottom: 1px solid #DDD !important;
      border-radius: 0;
      color: #777;
      background-color: #FFF !important;

      @at-root .theme--dark & {
        background-color: #222 !important;
        border-right-color: #444 !important;
        border-bottom-color: #444 !important;
        color: #CCC;
      }

      .v-icon {
        color: mc('blue', '700');
      }

      &:first-child {
        border-top-left-radius: 5px;
        border-bottom-left-radius: 5px;
      }

      &:last-child {
        border-top-right-radius: 5px;
        border-bottom-right-radius: 5px;
      }
    }
  }
}

</style>
