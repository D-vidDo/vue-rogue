<template>
  <!-- <v-btn @click="openSnackbar">Show snackbar</v-btn> -->
  <div>
  </div>
</template>

<script>
  import { showSnackbar } from '/GlobalActions';
  export default {
    methods: {
      openSnackbar: () => {
        showSnackbar('Hello from snackbar!');
      },
    },
  };
</script>