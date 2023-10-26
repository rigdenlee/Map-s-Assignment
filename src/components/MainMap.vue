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

    this.map.addControl(new mapboxgl.FullscreenControl());
    this.loadPointDataLayer(); // Load the Point data layer
    this.loadBorderLayer(); // Load the Polygon data layer
    this.loadPolygonDataLayer();
  },
  methods: {
    async loadPointDataLayer() {
      try {
        const response = await fetch("http://localhost:3000/us-churches");
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        const data = await response.json();

        for (const obj of data) {
          if (obj.LONGITUDE !== undefined && obj.LATITUDE !== undefined) {
            const popup = new mapboxgl.Popup({ offset: 25 }).setHTML(
              `<h3 class="description-title">Church Details:</h3>
              <p class="description-detail">Name: ${obj.NAME}</p>
              <p class="description-detail">ATTENDANCE: ${obj.ATTENDANCE}</p>
              <p class="description-detail">City: ${obj.CITY}</p>
              <p class="description-detail">County: ${obj.COUNTY}</p>
              <p class="description-detail">State: ${obj.STATE}</p>`
            );

            new mapboxgl.Marker({
              color: "black",
              rotationAlignment: "horizon",
            })
              .setLngLat([obj.LONGITUDE, obj.LATITUDE])
              .setPopup(popup)
              .addTo(this.map);
          }
        }
      } catch (error) {
        console.error("There was a problem with the fetch operation:", error);
      }
    },

    async loadPolygonDataLayer() {
      let hoveredPolygonId = null;
      try {
        const response = await fetch(
          "https://docs.mapbox.com/mapbox-gl-js/assets/us_states.geojson"
        );

        if (!response.ok) {
          throw new Error("Network response was not ok");
        }

        const data = await response.json();

        this.map.on("load", () => {
          this.map.addSource("states", {
            type: "geojson",
            data: data,
          });

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

          this.map.on("mouseleave", "state-fills", () => {
            if (hoveredPolygonId !== null) {
              this.map.setFeatureState(
                { source: "states", id: hoveredPolygonId },
                { hover: false }
              );
            }
          });
        });
      } catch (error) {
        console.error("There was a problem with the fetch operation:", error);
      }
    },

    async loadBorderLayer() {
      try {
        const response = await fetch("http://localhost:3000/us-mexico-boarder");

        if (!response.ok) {
          throw new Error("Network response was not ok");
        }

        const data = await response.json();
        const featureList = data.map(el => {
          return {
            type: "Feature",
            properties: {
              name: el.properties.gen_type,
              color: el.properties.barrier === "fence" ? "#ff0000" : "#33FF57",
            },
            geometry: {
              type: "LineString",
              coordinates: el.geometry.coordinates,
            },
          };
        });

        const geoJson = {
          type: "FeatureCollection",
          features: featureList,
        };

        this.map.on("load", () => {
          this.map.addSource("route", {
            type: "geojson",
            data: geoJson,
          });

          featureList.forEach((feature, index) => {
            this.map.addLayer({
              id: `route-${index}`,
              type: "line",
              source: "route",
              layout: {
                "line-join": "round",
                "line-cap": "round",
              },
              paint: {
                "line-color": feature.properties.color,
                "line-width": 20,
              },
              filter: ["==", "$id", index], // Filter by feature index
            });
          });
        });
      } catch (error) {
        console.error("There was a problem with the fetch operation:", error);
      }
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

.description-title {
  text-align: left;
}

.description-detail {
  text-align: left;
}
</style>
