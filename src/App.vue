<template>
  <div id="map"></div>
</template>

<script lang="ts">
import { defineComponent, onMounted } from 'vue';

import Map from 'ol/Map';
import { Tile as TileLayer, Vector as VectorLayer } from 'ol/layer';
import { Vector as VectorSource, OSM } from 'ol/source';
import { Style, Circle, Fill } from 'ol/style';
import View from 'ol/View';
import { fromLonLat, transformExtent } from 'ol/proj';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';

export default defineComponent({
  name: 'App',
  setup() {
    onMounted(() => {
      const source = new VectorSource();
      const map = new Map({
        layers: [
          new TileLayer({
            source: new OSM()
          }),
          new VectorLayer({
            source: source,
            style: new Style({
              image: new Circle({
                radius: 5.0,
                fill: new Fill({
                  color: 'rgba(0, 0, 255, 0.5)'
                })
              })
            })
          })
        ],
        target: 'map',
        view: new View({
          center: fromLonLat([136.89826501782872, 35.17743968529702]),
          zoom: 15.0,
          minZoom: 5.0,
          maxZoom: 20.0
        })
      });

      let intervalID: number;

      map.addEventListener('movestart', () => {
        clearInterval(intervalID);
      });

      map.addEventListener('moveend', () => {
        intervalID = setInterval(() => {
          const extent = transformExtent(map.getView().calculateExtent(map.getSize()), 'EPSG:3857', 'EPSG:4326');

          source.addFeature(new Feature({
            geometry: new Point(fromLonLat([
              getRandom(extent[0], extent[2]),
              getRandom(extent[1], extent[3])
            ]))
          }));
        }, 1000);
      });
    });

    function getRandom(min: number, max: number): number {
      return Math.random() * (max - min) + min;
    }
  }
});
</script>

<style>
html {
  height: 100%;
}

body {
  height: 100%;
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100%;
}

#map {
  height: 100%;
}
</style>
