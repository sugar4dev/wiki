<template>  
  <v-footer class="justify-center" :color="bgColor" inset>
    <div class="caption grey--text" :class="$vuetify.theme.dark ? `text--lighten-1` : `text--darken-1`">
      <template v-if="footerOverride"><span v-html="footerOverrideRender + ` |&nbsp;`"></span></template>
      <template v-else-if="company && company.length > 0 && contentLicense !== ``"><span v-if="contentLicense === `alr`">{{ $t('common:footer.copyright', { company: company, year: currentYear, interpolation: { escapeValue: false } }) }} |&nbsp;</span><span v-else>{{ $t('common:footer.license', { company: company, license: $t('common:license.' + contentLicense), interpolation: { escapeValue: false } }) }} |&nbsp;</span></template><span>{{ $t('common:footer.poweredBy') }} <a href="https://wiki.js.org" ref="nofollow">Wiki.js</a></span>
    </div>
  </v-footer>
</template>

<script>
import { get } from 'vuex-pathify'
import MarkdownIt from 'markdown-it'

const md = new MarkdownIt({
  html: false,
  breaks: false,
  linkify: true
})

export default {
  props: {
    color: {
      type: String,
      default: 'grey lighten-3'
    },
    darkColor: {
      type: String,
      default: 'grey darken-3'
    }
  },
  data() {
    return {
      currentYear: (new Date()).getFullYear()
    }
  },
  computed: {
    company: get('site/company'),
    contentLicense: get('site/contentLicense'),
    footerOverride: get('site/footerOverride'),
    footerOverrideRender () {
      if (!this.footerOverride) { return '' }
      return md.renderInline(this.footerOverride)
    },
    bgColor() {
      if (!this.$vuetify.theme.dark) {
        return this.color
      } else {
        return this.darkColor
      }
    }
  }
}
</script>

<style lang="scss">
  .v-footer {
    a {
      text-decoration: none;
    }

    &.altbg {
      background: mc('theme', 'primary');

      span {
        color: mc('blue', '300');
      }

      a {
        color: mc('blue', '200');
      }
    }
  }
</style>
