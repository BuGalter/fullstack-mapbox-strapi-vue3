<template>
  <AppTitle :titleText="titleText" />
  <AppMap
    :token="accessToken"
    :mapCenter="mapCenter"
    :colors="colors"
    :zoom="zoom"
    :isClicked="isClicked"
    :addPoint="addPoint"
    :baseUrl="baseUrl"
    :key="reloadMapKey"
  />
  <div class="conteiner">
    <AppButton
      class="button"
      :buttonText="buttonAddText"
      :eventHandler="this.clicked"
    />
    <div class="inputArea">
      <span v-for="point in coordinatePoint" :key="point">{{ point }}</span>
    </div>
    <AppButton
      class="button"
      :buttonText="buttonAbortText"
      :eventHandler="this.abortSend"
    />
    <AppButton
      class="button"
      :buttonText="buttonSendText"
      :eventHandler="this.sendData"
    />
  </div>
</template>

<script>
import axios from 'axios';
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
      buttonAbortText: 'Отменить ввод',
      alertText: `Для выбора границ участка\nНеобходимо нажимать на карту в нужных точках\nИх должно быть не менее трех`,
      alertTextNeedData: 'Не хватает координат!',
      accessToken: process.env.VUE_APP_MAPBOX_TOKEN,
      baseUrl: 'http://localhost:1337/api',
      urlAddPlots: '/plots',
      urlAddPoints: '/points',
      mapCenter: [43.278971, 19.375222],
      zoom: 2,
      isClicked: false,
      pointsToSave: [],
      reloadMapKey: false,
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
      /**
       * Меняет состояние кнопки - добавить участок
       */
      console.log(event);
      alert(this.alertText);
      this.isClicked = true;
    },

    sendData: function (event) {
      /**
       * Проверка перед отправкой, что точек не менее трех
       * Вызов функции для отправки данных к апи
       */
      console.log(event);
      if (this.coordinatePoint.length < 3) {
        alert(this.alertTextNeedData);
      } else {
        this.axiosSendData();
        this.coordinatePoint = [];
        this.isClicked = false;
      }
    },

    abortSend: function (event) {
      /**
       * Отмена отправки, очистка поля в котором отображаются текущие
       * координаты точек для отправки
       */
      console.log(event);
      this.isClicked = false;
      this.coordinatePoint = [];
    },

    addPoint: function (point) {
      /**
       * Добавляет точки для отправки
       */
      this.coordinatePoint.push(point);
    },

    generatePointsToSave: function () {
      /**
       * Приводит точки к виду, который необходим для сохранения
       * координат в базе данных
       */
      this.pointsToSave = this.coordinatePoint.map((element) => {
        return {
          latitude: element[0],
          longitude: element[1],
        };
      });
    },

    addPlotToDb: async function () {
      /**
       * Добавляет в базу данных введеный пользователем участок
       */
      try {
        let response = await axios.post('http://localhost:1337/api/plots', {
          data: {
            level: Math.floor(Math.random() * 10),
          },
        });

        return response.data.data.id;
      } catch (error) {
        alert(`Упс... Что то пошло не так!!! Участок не добавлен!!!`);
        console.error(error);
      }
    },

    addPointToDb: async function (point) {
      /**
       * Добавляет в базу данных точку из массива добовляемых координат
       */
      try {
        const response = await axios.post('http://localhost:1337/api/points', {
          data: point,
        });

        return response.data.data.id;
      } catch (error) {
        alert(`Упс... Что то пошло не так!!! Точка не добавленна!!!`);
        console.error(error);
      }
    },

    addConnectionPlotPoint: async function (pointId, plotId) {
      /**
       * Добавляет в базу данных связь между участком и текущей точкой
       */
      try {
        await axios.put(`http://localhost:1337/api/plots/${plotId}`, {
          data: {
            points: {
              connect: [pointId],
            },
          },
        });
      } catch (error) {
        alert(`Упс... Что то пошло не так!!! Связь не была созданна!!!`);
        console.error(error);
      }
    },

    axiosSendData: async function () {
      /**
       * Основная функция отправки заданого пользователем участка в базу данных
       */
      this.generatePointsToSave();

      let plotId = await this.addPlotToDb();

      this.pointsToSave.forEach(async (point) => {
        let pointId = await this.addPointToDb(point);
        await this.addConnectionPlotPoint(pointId, plotId);
      });

      alert(`Участок ${plotId} добавлен!`);
      this.reloadMapKey = !this.reloadMapKey;
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
  text-align: center;
  font-size: small;
}

.button {
  align-self: center;
  padding-top: 10px;
  padding-bottom: 10px;
}
</style>
