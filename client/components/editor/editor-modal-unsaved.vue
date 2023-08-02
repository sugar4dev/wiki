<template>  
  <v-dialog v-model="isShown" max-width="550">
    <v-card>
      <div class="dialog-header is-short is-red">
        <v-icon class="mr-2" color="white">mdi-alert</v-icon><span>{{$t('editor:unsaved.title')}}</span>
      </div>
      <v-card-text class="pt-4">
        <div class="body-2">{{$t('editor:unsaved.body')}}</div>
      </v-card-text>
      <v-card-chin>
        <v-spacer></v-spacer>
        <v-btn text @click="isShown = false">{{$t('common:actions.cancel')}}</v-btn>
        <v-btn class="px-4" color="red" @click="discard" dark>{{$t('common:actions.discardChanges')}}</v-btn>
      </v-card-chin>
    </v-card>
  </v-dialog>
</template>

<script>

export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return { }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    }
  },
  methods: {
    async discard() {
      this.isShown = false
      this.$emit('discard', true)
    }
  }
}
</script>
