<template>
  <h2 class="title">
    {{ titleText }}
  </h2>
</template>

<script>
export default {
  name: 'AppTitle',

  props: ['titleText'],
};
</script>

<style>
.title {
  margin-top: 0;
  text-align: center;
}
</style>
