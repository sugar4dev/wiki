<template>  
  <div>
    <v-divider class="my-0"></v-divider>
    <v-card-actions :class="$vuetify.theme.dark ? 'grey darken-4-l5' : 'grey lighten-4'">
      <slot></slot>
    </v-card-actions>
  </div>
</template>

<script>
export default { }
</script>
