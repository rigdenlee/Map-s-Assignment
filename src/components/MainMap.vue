<template>
  <div>
    <div class="content">
      <div>
        <h2>MAP USING MAPBOX GL</h2>
        <h2>MAP USING MAPBOX GL</h2>
      </div>
    </div>
    <div :style="{ display: 'flex', justifyContent: 'center' }">
      <div id="map"></div>
      <div class="card" v-if="selectedFeature">
        <div class="card-body">
          <h5 class="card-title">Details</h5>
          <img
            class="card-cross"
            src="../assets/cross.svg"
            alt="cross"
            width="30px"
          />
          <p class="card-text">{{ selectedFeature }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import mapboxgl from "mapbox-gl";
import "mapbox-gl/dist/mapbox-gl.css";

export default {
  data() {
    return {
      map: null,
      selectedFeature: null,
    };
  },
  mounted() {
    mapboxgl.accessToken = process.env.VUE_APP_MAPBOX_ACCESS_KEY;

    this.map = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/mapbox/streets-v12",
      center: [-101.299591, 47.116386],
      zoom: 3,
    });

    this.loadPointDataLayer(); // Load the Point data layer
    this.loadPolygonDataLayer(); // Load the Polygon data layer

    // Add click event listener to the map
    this.map.on("click", this.onMapClick);
    fetch("http://localhost:3000/us-churches")
    .then((response) => {
      // Check if the response status is OK (status code 200)
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      return response.json();
    })
    .then((data) => {
      // Use the obtained coordinates as needed
      for (const obj of data) {
        if (obj.LONGITUDE != undefined && obj.LATITUDE != undefined) {
          new mapboxgl.Marker({ color: "black", rotationAlignment: "horizon" })
            .setLngLat([obj.LONGITUDE, obj.LATITUDE])
            .addTo(this.map);
        }
      }
    })
    .catch((error) => {
      console.error("There was a problem with the fetch operation:", error);
    });
  },
  methods: {
    loadPointDataLayer() {
      // Fetch and add Point data to the map
      // Replace with your API call and rendering logic
    },
    loadPolygonDataLayer() {
      // Fetch and add Polygon data to the map
      let hoveredPolygonId = null;

      this.map.on("load", () => {
        this.map.addSource("states", {
          type: "geojson",
          data: "https://docs.mapbox.com/mapbox-gl-js/assets/us_states.geojson",
        });

        // The feature-state dependent fill-opacity expression will render the hover effect
        // when a feature's hover state is set to true.
        this.map.addLayer({
          id: "state-fills",
          type: "fill",
          source: "states",
          layout: {},
          paint: {
            "fill-color": "#627BC1",
            "fill-opacity": [
              "case",
              ["boolean", ["feature-state", "hover"], false],
              1,
              0.5,
            ],
          },
        });

        this.map.addLayer({
          id: "state-borders",
          type: "line",
          source: "states",
          layout: {},
          paint: {
            "line-color": "#627BC1",
            "line-width": 2,
          },
        });

        // When the user moves their mouse over the state-fill layer, we'll update the
        // feature state for the feature under the mouse.
        this.map.on("mousemove", "state-fills", (e) => {
          if (e.features.length > 0) {
            if (hoveredPolygonId !== null) {
              this.map.setFeatureState(
                { source: "states", id: hoveredPolygonId },
                { hover: false }
              );
            }
            hoveredPolygonId = e.features[0].id;
            this.map.setFeatureState(
              { source: "states", id: hoveredPolygonId },
              { hover: true }
            );
          }
        });

        // When the mouse leaves the state-fill layer, update the feature state of the
        // previously hovered feature.
        this.map.on("mouseleave", "state-fills", () => {
          if (hoveredPolygonId !== null) {
            this.map.setFeatureState(
              { source: "states", id: hoveredPolygonId },
              { hover: false }
            );
          }
          hoveredPolygonId = null;
        });
      });
      // Replace with your API call and rendering logic
    },
    onMapClick(e) {
      this.selectedFeature = `Clicked at Longitude: ${e.lngLat.lng}, Latitude: ${e.lngLat.lat}`;
    },
  },
};
</script>

<style>
.content {
  display: flex;
  justify-content: center;
  align-items: center;
  /* position: relative;
  top: 9vh; */
  width: 100%;
  height: 14vh;
}

.content h2 {
  word-wrap: break-word;
  color: #fff;
  font-size: 5em;
  position: absolute;
  transform: translate(-50%, -50%);
}

.content h2:nth-child(1) {
  color: white;
  -webkit-text-stroke: 2px #03a9f4;
}

.content h2:nth-child(2) {
  color: #03a9f4;
  animation: animate 4s ease-in-out infinite;
}

@keyframes animate {
  0%,
  100% {
    clip-path: polygon(
      0% 45%,
      16% 44%,
      33% 50%,
      54% 60%,
      70% 61%,
      84% 59%,
      100% 52%,
      100% 100%,
      0% 100%
    );
  }

  50% {
    clip-path: polygon(
      0% 60%,
      15% 65%,
      34% 66%,
      51% 62%,
      67% 50%,
      84% 45%,
      100% 46%,
      100% 100%,
      0% 100%
    );
  }
}

#map {
  text-align: center;
  height: 82vh;
  width: 76vw;
}

.card {
  width: 300px;
  position: absolute;
  top: 10px;
  right: 0px;
  z-index: 1;
  padding: 10px;
  background: white;
  transform: translateY(-0.5%);
  box-shadow: 0 4rem 8rem rgba(255, 255, 255, 0.2);
}

.card-body {
  display: flex;
  flex-direction: row;
}

.card-title {
  width: 80%;
}

.card-cross {
  width: 20%;
}
</style>
