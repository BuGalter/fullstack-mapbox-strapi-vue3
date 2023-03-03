<template>
  <div class="conteiner">
    <h2 class="title">
      {{ text }}
    </h2>
  </div>
</template>

<script>
export default {
  name: 'AppTitle',
  props: {
    text: String,
  },
};
</script>

<style>
.conteiner {
  margin-top: 0;
  padding-top: 0;
}

.title {
  text-align: center;
}
</style>
