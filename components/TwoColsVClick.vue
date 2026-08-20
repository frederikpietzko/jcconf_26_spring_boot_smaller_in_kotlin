<script setup>

</script>

<template>
  <div class="grid grid-cols-2 gap-4">
    <div>
      <v-clicks>
        <slot name="left" />
      </v-clicks>
    </div>

    <div>
      <v-clicks>
        <slot name="right" />
      </v-clicks>
    </div>
  </div>

</template>

<style scoped>

</style>