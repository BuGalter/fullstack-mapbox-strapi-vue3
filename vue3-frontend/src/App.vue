<template>
  <AppTitle :titleText="titleText" />
  <AppMap
    :token="accessToken"
    :mapCenter="mapCenter"
    :colors="colors"
    :zoom="zoom"
    :isClicked="isClicked"
    :addPoint="addPoint"
  />
  <div class="conteiner">
    <AppButton class="button" :buttonText="buttonAddText" :eventHandler="this.clicked" />
    <div class="inputArea">
      <span v-for="point in coordinatePoint" :key="point">{{ point }}</span>
    </div>
    <AppButton class="button" :buttonText="buttonSendText" :eventHandler="this.sendData" />
  </div>
</template>

<script>
import AppButton from './components/AppButton.vue';
import AppTitle from './components/AppTitle.vue';
import AppMap from './components/AppMap.vue';

export default {
  name: 'App',
  components: {
    AppTitle,
    AppMap,
    AppButton,
  },

  data() {
    return {
      titleText: 'Приложение для отображения участков на карте',
      buttonAddText: 'Добавить участок',
      buttonSendText: 'Сохранить участок',
      alertText: `Для выбора границ участка\nНеобходимо нажимать на карту в нужных токах\nИх должно быть не менее трех`,
      accessToken: process.env.VUE_APP_MAPBOX_TOKEN,
      mapCenter: [43.278971, 19.375222],
      zoom: 2,
      isClicked: false,
      colors: [
        '#00008B',
        '#9400D3',
        '#00FF00',
        '#B8860B',
        '#00FF00',
        '#F0E68C',
      ],
      coordinatePoint: [],
    };
  },

  methods: {
    clicked: function (event) {
      console.log(event);
      alert(this.alertText);
      this.isClicked = true;
    },

    sendData: function (event) {
      console.log(event);
      this.isClicked = false;
      this.coordinatePoint = [];
    },

    addPoint: function (point) {
      this.coordinatePoint.push(point);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 90vw;
  margin-left: auto;
  margin-right: auto;
}

.conteiner {
  display: flex;
  justify-content: flex-start;
}

.inputArea {
  width: 60%;
  align-self: center;
}

.button {
  align-self: center;
  padding-top: 10px;
  padding-bottom: 10px;
}
</style>
