<template>
  <div id="mapConteiner" />
</template>

<script>
import axios from 'axios';
import 'mapbox-gl/dist/mapbox-gl.css';
import mapboxgl from 'mapbox-gl';

export default {
  name: 'AppMap',

  props: [
    'token',
    'mapCenter',
    'colors',
    'zoom',
    'isClicked',
    'addPoint',
    'addMarker',
    'baseUrl',
    'deletePlot',
  ],

  data() {
    return {
      plots: null,
      points: null,
      urlGetPlots: this.baseUrl + '/plots?populate=*',
      mapStyle: 'mapbox://styles/mapbox/outdoors-v11',
      alertText: 'Упс... Апи молчит!!!',
      plotOnDelete: null,
    };
  },

  methods: {
    getPlotsFromApi: async function () {
      /**
       * Получить данные об участках из апи
       */
      try {
        let response = await axios.get(this.urlGetPlots);
        return response.data.data;
      } catch (error) {
        alert(this.alertText);
        console.error(error);
      }
    },

    getGeoJsonData(points) {
      /**
       * Преобразовать полученные данные из апи к ГеоДжейсон формату
       */
      const geoJsonData = points.map((element) => {
        return [element.attributes.longitude, element.attributes.latitude];
      });

      return geoJsonData;
    },

    randomInteger(min, max) {
      /**
       * Получить рендомное целое число из интервала 0, длинна массива цветов,
       * чтобы выбрать цвет для закрашивания текущего участка
       */
      let rand = min + Math.random() * (max - min);
      return Math.floor(rand);
    },

    async handlerButtonDelete() {
      /**
       * Подтверждает удаления заданнго участка
       */
      await this.deletePlot(this.plotOnDelete);
    },

    createPopUpTextElement(text, className) {
      /**
       * Создает контейнер для текста всплывающего меню
       */
      let p = document.createElement('p');
      p.textContent = text;
      p.className = className;

      return p;
    },

    createPopUpButton(text, className, handler) {
      /**
       * Создает кнопку для всплывающего окна
       */
      let button = document.createElement('button');
      button.type = 'button';
      button.className = className;
      button.textContent = text;
      button.onclick = handler;

      return button;
    },

    createMap() {
      /**
       * Создает экземпляр карты
       */
      const map = new mapboxgl.Map({
        container: 'mapConteiner',
        style: this.mapStyle,
        center: this.mapCenter,
        zoom: this.zoom,
      });

      return map;
    },
  },

  mounted() {
    /**
     * Основная функция, где происходит:
     * Получение экземпляра карты
     * Получение данных об участках из апи
     * Отрисовка карты
     * Отрисовка участков в соответствие с данными из апи
     * Добавление навигации по карте
<<<<<<< HEAD
     * Добавление маркера при нажатие на карту в режиме ввода данных об участке
=======
     * Добавление всплывающего меню
     * Добавление маркеров - границ участка  при нажатие на карту
     * в режиме ввода данных об участке
>>>>>>> develop
     */
    mapboxgl.accessToken = this.token;

    const map = this.createMap();

    map.on('load', async () => {
      const nav = new mapboxgl.NavigationControl();
      map.addControl(nav, 'top-right');

      this.plots = await this.getPlotsFromApi();

      for (let i = 0; i < this.plots.length; i += 1) {
        this.points = this.plots[i].attributes.points.data;

        let coordinates = this.getGeoJsonData(this.points);

        let plotId = this.plots[i].id.toString();

        let numberRandomColor = this.randomInteger(0, this.colors.length);

        map.addSource(plotId, {
          /**
           * Добавление участка по координатам из апи
           */
          type: 'geojson',
          data: {
            type: 'Feature',
            geometry: {
              type: 'Polygon',
              coordinates: [coordinates],
            },
          },
        });

        map.addLayer({
          /**
           * Закрашивание участка
           */
          id: plotId,
          type: 'fill',
          source: plotId, // reference the data source
          layout: {},
          paint: {
            'fill-color': this.colors[numberRandomColor], // random color fill
            'fill-opacity': 0.7,
          },
        });

        map.on('click', plotId, (e) => {
          /**
           * Создание всплывающего меню
           */
          this.plotOnDelete = e.features[0].layer.id;
          let div = document.createElement('div');
          let text = `Участок номер: ${plotId}`;
          let p = this.createPopUpTextElement(text, 'plot');
          div.append(p);
          let button = this.createPopUpButton(
            'Удалить',
            'button',
            this.handlerButtonDelete
          );
          div.append(button);
          new mapboxgl.Popup()
            .setLngLat(e.lngLat)
            .setDOMContent(div)
            .addTo(map);
        });

        // Change the cursor to a pointer when
        // the mouse is over the plots layer.
        map.on('mouseenter', plotId, () => {
          map.getCanvas().style.cursor = 'pointer';
        });

        // Change the cursor back to a pointer
        // when it leaves the plots layer.
        map.on('mouseleave', plotId, () => {
          map.getCanvas().style.cursor = '';
        });
      }
    });

    map.on('click', (e) => {
      /**
       * Создание маркеров, ограничевающих, выбраный участок
       */
      let point = [e.lngLat.lat, e.lngLat.lng];
      if (this.isClicked) {
        let marker = new mapboxgl.Marker();
        marker.setLngLat([point[1], point[0]]).addTo(map);
        this.addMarker(marker);
        this.addPoint(point);
      }
    });
  },
};
</script>

<style>
#mapConteiner {
  height: 80vh;
}

.plot {
  text-align: center;
}
</style>
