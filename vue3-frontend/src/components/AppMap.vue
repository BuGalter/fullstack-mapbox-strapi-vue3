<template>
  <div id="mapConteiner" />
</template>

<script>
import axios from 'axios';
import 'mapbox-gl/dist/mapbox-gl.css';
import mapboxgl from 'mapbox-gl';

export default {
  name: 'AppMap',

  props: ['token', 'mapCenter', 'colors', 'zoom', 'isClicked', 'addPoint', 'baseUrl'],

  data() {
    return {
      plots: null,
      points: null,
      urlGetPlots: this.baseUrl + '/plots?populate=*',
      marker: new mapboxgl.Marker(),
      mapStyle: 'mapbox://styles/mapbox/outdoors-v11',
      alertText: 'Упс... Апи молчит!!!',
    };
  },

  methods: {
    getPlotsFromApi: async function () {
      try {
        let response = await axios.get(this.urlGetPlots);
        return response.data.data;
      } catch (error) {
        alert(this.alertText);
        console.error(error);
      }
    },

    getGeoJsonData(points) {
      const geoJsonData = points.map((element) => {
        return [element.attributes.longitude, element.attributes.latitude];
      });

      return geoJsonData;
    },

    randomInteger(min, max) {
      // случайное число от min до (max+1)
      let rand = min + Math.random() * (max + 1 - min);
      return Math.floor(rand);
    },
  },

  mounted() {
    mapboxgl.accessToken = this.token;

    const map = new mapboxgl.Map({
      container: 'mapConteiner',
      style: this.mapStyle,
      center: this.mapCenter,
      zoom: this.zoom,
    });

    map.on('load', async () => {
      this.plots = await this.getPlotsFromApi();

      for (let i = 0; i < this.plots.length; i += 1) {
        this.points = this.plots[i].attributes.points.data;
        let coordinates = this.getGeoJsonData(this.points);
        let plotId = this.plots[i].id.toString();
        let numberRandomColor = this.randomInteger(0, this.colors.length - 1);

        map.addSource(plotId, {
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
          id: plotId,
          type: 'fill',
          source: plotId, // reference the data source
          layout: {},
          paint: {
            'fill-color': this.colors[numberRandomColor], // random color fill
            'fill-opacity': 0.7,
          },
        });
      }
    });

    const nav = new mapboxgl.NavigationControl();
    map.addControl(nav, 'top-right');

    map.on('click', (e) => {
      let point = [e.lngLat.lat, e.lngLat.lng];
      this.marker.remove();
      if (this.isClicked) {
        this.marker.remove();
        this.marker.setLngLat([point[1], point[0]]).addTo(map);
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
</style>
