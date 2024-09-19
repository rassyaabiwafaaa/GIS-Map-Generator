<template>
  <div class="app-container">
    <!-- Sidebar with Shadcn UI -->
    <aside class="sidebar">
      <h2 class="title">Layer Controls</h2>
      <div class="control-buttons">
        <button @click="toggleLayer(1)" :class="{ active: layerStates.layer1Active, inactive: !layerStates.layer1Active }">Check Point 1</button>
        <button @click="toggleLayer(2)" :class="{ active: layerStates.layer2Active, inactive: !layerStates.layer2Active }">Check Point 2</button>
      </div>
      <div v-if="error" class="error-message">{{ error }}</div>
    </aside>
    <!-- Map Container -->
    <div id="map"></div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from "vue";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  name: "MapComponent",
  setup() {
    const map = ref(null);
    const layers = reactive({
      layer1: null,
      layer2: null,
    });
    const layerStates = reactive({
      layer1Active: true,
      layer2Active: true,
    });
    const error = ref(null);

    // Dummy GeoJSON Data
    const geoJSONLayer1 = {
      type: "FeatureCollection",
      features: [
        {
          type: "Feature",
          properties: {
            title: "Layer 1 - Point 1",
            description: "This is the first point in Layer 1.",
          },
          geometry: {
            type: "Point",
            coordinates: [106.8456, -6.2088],
          },
        },
      ],
    };

    const geoJSONLayer2 = {
      type: "FeatureCollection",
      features: [
        {
          type: "Feature",
          properties: {
            title: "Layer 2 - Point 1",
            description: "This is the first point in Layer 2.",
          },
          geometry: {
            type: "Point",
            coordinates: [106.865, -6.22],
          },
        },
      ],
    };

    const initializeMap = () => {
      try {
        map.value = L.map("map").setView([-6.2088, 106.8456], 6);

        L.tileLayer("https://tile.openstreetmap.org/{z}/{x}/{y}.png", {
          maxZoom: 19,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>',
        }).addTo(map.value);

        // Initialize Layers
        layers.layer1 = L.geoJSON(geoJSONLayer1, {
          onEachFeature: (feature, layer) => {
            if (feature.properties && feature.properties.title) {
              layer.bindPopup(`<h3>${feature.properties.title}</h3><p>${feature.properties.description}</p>`);
            }
          },
        }).addTo(map.value);

        layers.layer2 = L.geoJSON(geoJSONLayer2, {
          onEachFeature: (feature, layer) => {
            if (feature.properties && feature.properties.title) {
              layer.bindPopup(`<h3>${feature.properties.title}</h3><p>${feature.properties.description}</p>`);
            }
          },
        }).addTo(map.value);
      } catch (err) {
        error.value = "Failed to initialize map: " + err.message;
      }
    };

    const toggleLayer = (layerNumber) => {
      try {
        if (layerNumber === 1) {
          if (layerStates.layer1Active) {
            map.value.removeLayer(layers.layer1);
          } else {
            layers.layer1.addTo(map.value);
          }
          layerStates.layer1Active = !layerStates.layer1Active;
        } else if (layerNumber === 2) {
          if (layerStates.layer2Active) {
            map.value.removeLayer(layers.layer2);
          } else {
            layers.layer2.addTo(map.value);
          }
          layerStates.layer2Active = !layerStates.layer2Active;
        }
      } catch (err) {
        error.value = "Failed to toggle layer: " + err.message;
      }
    };

    onMounted(() => {
      initializeMap();
    });

    return {
      layerStates,
      toggleLayer,
      error,
    };
  },
};
</script>

<style>
body {
  margin: 0;
  overflow: hidden;
}

.app-container {
  display: flex;
  height: 100vh;
  width: 100vw;
}

#map {
  flex: 1;
  height: 100%;
  width: 100%;
}

.sidebar {
  width: 200px;
  background: #f8f9fa;
  padding: 20px;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.title {
  font-size: 24px;
  font-family: Arial, Helvetica, sans-serif;
  color: black;
}

.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.control-buttons button {
  padding: 10px;
  background-color: #3f3e3e;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s, opacity 0.3s;
}

.control-buttons button.active {
  background-color: #28a745;
}

.control-buttons button.inactive {
  background-color: #6c757d;
  opacity: 0.6;
}

.control-buttons button:hover {
  background-color: #0056b3;
}

.error-message {
  color: red;
  font-weight: bold;
}
</style>
