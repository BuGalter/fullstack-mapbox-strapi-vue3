<template>
  <div class="conteiner">
    <AddButton />
    <h2 class="title">
      {{ text }}
    </h2>
  </div>
</template>

<script>
import AddButton from './AddButton.vue';

export default {
  name: 'AppTitle',

  components: {
    AddButton,
  },

  data() {
    return {
      text: 'Приложение для отображения участков на карте',
    };
  },
};
</script>

<style>
.conteiner {
  display: flex;
  justify-content: space-between;
  margin-top: 0;
  padding-top: 0;
  width: 90vw;
  margin-left: auto;
  margin-right: auto;
}

.title {
  align-self: center;
}
</style>
