<template>
  <AppTitle />
  <div id="mapConteiner" />
</template>

<script>
import mapboxgl from 'mapbox-gl';
import AppTitle from './components/AppTitle.vue';
import axios from 'axios';
import 'mapbox-gl/dist/mapbox-gl.css';

export default {
  name: 'App',
  components: {
    AppTitle,
  },

  data() {
    return {
      accessToken: process.env.VUE_APP_MAPBOX_TOKEN,
      plots: null,
      points: null,
      mapCenter: [43.278971, 19.375222],
      colors: [
        '#00008B',
        '#9400D3',
        '#00FF00',
        '#B8860B',
        '#00FF00',
        '#F0E68C',
      ],
    };
  },

  methods: {
    fetchPlots() {
      axios
        .get('http://localhost:1337/api/plots?populate=*')
        .then((response) => {
          this.plots = response.data.data;
        })
        .catch((err) => console.log(err));
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
    this.fetchPlots();

    mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_TOKEN;

    const map = new mapboxgl.Map({
      container: 'mapConteiner',
      style: 'mapbox://styles/mapbox/outdoors-v11',
      center: this.mapCenter,
      zoom: 1,
    });

    map.on('load', () => {
      for (let i = 0; i < this.plots.length; i += 1) {
        this.points = this.plots[i].attributes.points.data;
        let coordinates = this.getGeoJsonData(this.points);
        let plotId = this.plots[i].id.toString();
        let numberRandomColor = this.randomInteger(0, this.plots.length); 

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
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#mapConteiner {
  height: 90vh;
  width: 90vw;
  margin-left: auto;
  margin-right: auto;
}
</style>
