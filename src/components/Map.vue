<script setup>
import { onMounted, defineProps, watch } from 'vue';
import {FeatureGroup, Map, TileLayer, GeoJSON, Marker, Popup} from 'leaflet';
import 'leaflet/dist/leaflet.css'
import weatherLocations from '@/weatherLocations';

const { currentSelectedFiringTime, activeLayers } = defineProps(['currentSelectedFiringTime', 'activeLayers']);

watch(
  () => currentSelectedFiringTime,
  () => {
    if (currentSelectedFiringTime) {
      updateFiringRangeToReflectCurrentFiringTime();
    } else {
      console.error('map nothing selected');
    }
  },
  {
    deep: true
  }
);

watch(
  () => activeLayers.campingArea,
  (newValue) => {
    if(newValue) {
      addLayerCampingArea();
      return;
    }
    removeLayerCampingArea();
  }
);

watch(
  () => activeLayers.nestingBirdsArea,
  (newValue) => {
    if(newValue) {
      addLayerGroundNestingBirds();
      return;
    }
    removeLayerGroundNestingBirds();
  }
);

watch(
  () => activeLayers.accessAreas,
  (newValue) => {
    if(newValue) {
      addLayerAccessLand();
      return;
    }
    removeLayerAccessLand();
  }
);

watch(
  () => activeLayers.weatherPoints,
  (newValue) => {
    if(newValue) {
      addLayerWeatherPoints();
      return;
    }
    removeLayerWeatherPoints();
  }
);

const endPoints = {
  campingArea: './data/dartmoor-camping-areas.geojson',
  accessLand: './data/dartmoor-access-land-areas.geojson',
  firingArea: './data/dartmoor-live-firing-areas.geojson',
  nestingArea: './data/dartmoor-ground-nesting-birds-areas.geojson',
  nationalParkArea: './data/dartmoor-national-park-outline.geojson',
}

let map = null;
const firingAreasLayer = new FeatureGroup();
const viewableFiringAreasLayer = new FeatureGroup();
const nationalParkAreaLayer = new FeatureGroup();
const campingAreaLayer = new FeatureGroup();
const accessAreaLayer = new FeatureGroup();
const nestingBirdsLayer = new FeatureGroup();
const weatherLocationLayer = new FeatureGroup();

onMounted(() => {
  map = new Map('map').setView([50.5762, -3.89190], 10);

  const tiles = new TileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  viewableFiringAreasLayer.addTo(map);
  loadFiringAreaGeoData();
  loadAndAddNationalParkArea();
  loadCampingArea();
  loadAccessLand();
  loadGroundNestingBirds();
  loadWeatherLocations();
});

/**
 * Firing Areas
 */

async function loadFiringAreaGeoData() {
  try {
    const response = await fetch(endPoints.firingArea);
    if (!response.ok) {
        throw new Error('Unable to get MOD firing area geojson');
    }

    const json = await response.json();

    new GeoJSON(json, {
      onEachFeature: function(feature, layer) {
        //console.log(feature, layer);
      },
      style: function (feature) {
        return {
          "fill": true,
          "color": "#ce023f",
          "weight": 5,
        }
      },
      'interactive': false,
    }).addTo(firingAreasLayer);
  } catch (error) {
    console.error("Error:", error);
  }
}

function updateFiringRangeToReflectCurrentFiringTime() {
  removeAllViewableFiringAreasLayers();

  for(let range in currentSelectedFiringTime.value.ranges) {
    const firings = currentSelectedFiringTime.value.ranges[range];
    if (firings.day || firings.night) {
      showFiringRangeByName(range);
    }
  }
}

function showFiringRangeByName(name) {
  firingAreasLayer.getLayers()[0].eachLayer(function(layer) {
    if(layer.feature.properties.name.toLowerCase().includes(name)) {
        layer.addTo(viewableFiringAreasLayer);
    }
  })
}

function removeAllViewableFiringAreasLayers() {
  viewableFiringAreasLayer.clearLayers();
}

/**
 * National Park Area
 */

async function loadAndAddNationalParkArea() {
  try {
    const response = await fetch(endPoints.nationalParkArea);
    if (!response.ok) {
        throw new Error('Unable to get national park area geojson');
    }

    const json = await response.json();

    new GeoJSON(json, {
      onEachFeature: function(feature, layer) {
        //console.log(feature, layer);
      },
      style: function (feature) {
        return {
          "fill": false,
          "color": "#cec702",
          "weight": 5,
        }
      },
      'interactive': false,
    }).addTo(nationalParkAreaLayer);

    addLayerNationalParkArea();
  } catch (error) {
    console.error("Error:", error);
  }
}

function addLayerNationalParkArea() {
  nationalParkAreaLayer.addTo(map);
}

function removeLayerNationalParkArea() {
  map.removeLayer(nationalParkAreaLayer);
}

/**
 * Camping Areas
 */

async function loadCampingArea() {
  try {
    const response = await fetch(endPoints.campingArea);
    if (!response.ok) {
        throw new Error('Unable to get camping area geojson');
    }

    const json = await response.json();

    new GeoJSON(json, {
      onEachFeature: function(feature, layer) {
        //console.log(feature, layer);
      },
      style: function (feature) {
        return {
          "fill": true,
          "color": "#48a322",
          "weight": 2,
        }
      },
      'interactive': false,
    }).addTo(campingAreaLayer);

  } catch (error) {
    console.error("Error:", error);
  }
}

function addLayerCampingArea() {
  campingAreaLayer.addTo(map);
}

function removeLayerCampingArea() {
  map.removeLayer(campingAreaLayer);
}

/**
 * Access Land
 */

async function loadAccessLand() {
  try {
    const response = await fetch(endPoints.accessLand);
    if (!response.ok) {
        throw new Error('Unable to get access area geojson');
    }

    const json = await response.json();

    new GeoJSON(json, {
      onEachFeature: function(feature, layer) {
        //console.log(feature, layer);
      },
      style: function (feature) {
        return {
          "fill": true,
          "color": "#9e8256",
          "weight": 2,
        }
      },
      'interactive': false,
    }).addTo(accessAreaLayer);

  } catch (error) {
    console.error("Error:", error);
  }
}

function addLayerAccessLand() {
  accessAreaLayer.addTo(map);
}

function removeLayerAccessLand() {
  map.removeLayer(accessAreaLayer);
}

/**
 * Ground nesting birds
 */

async function loadGroundNestingBirds() {
  try {
    const response = await fetch(endPoints.nestingArea);
    if (!response.ok) {
        throw new Error('Unable to get ground nesting birds area geojson');
    }

    const json = await response.json();

    new GeoJSON(json, {
      onEachFeature: function(feature, layer) {
        //console.log(feature, layer);
      },
      style: function (feature) {
        return {
          "fill": true,
          "color": "#f9876d",
          "weight": 5,
        }
      },
      'interactive': false,
    }).addTo(nestingBirdsLayer);

  } catch (error) {
    console.error("Error:", error);
  }
}

function addLayerGroundNestingBirds() {
  nestingBirdsLayer.addTo(map);
}

function removeLayerGroundNestingBirds() {
  map.removeLayer(nestingBirdsLayer);
}

/**
 * Weather Locations
 */

function loadWeatherLocations() {
  weatherLocations.forEach((item) =>{
    new Marker([item.geometry.lat, item.geometry.lng]).bindPopup(item.locationName).addTo(weatherLocationLayer);
  });
}

function addLayerWeatherPoints() {
  weatherLocationLayer.addTo(map);
}

function removeLayerWeatherPoints() {
  map.removeLayer(weatherLocationLayer);
}

</script>

<template>
  <div id="map" class="h-full z-1 bg-amber-300"></div>
</template>
