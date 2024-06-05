<template>
  <div>
    <input
      type="text"
      v-model="query"
      @input="debouncedFetchSuggestions"
      placeholder="Nhập địa chỉ..."
    />
    <ul v-if="suggestions.length">
      <li
        v-for="suggestion in suggestions"
        :key="suggestion.place_id"
        @click="selectSuggestion(suggestion)"
      >
        {{ suggestion.display_name }}
      </li>
    </ul>
    <div id="map" style="height: 500px"></div>
  </div>
</template>

<script>
import axios from "axios";
import { debounce } from "vue-debounce";
import L from "leaflet";
import "leaflet/dist/leaflet.css";

export default {
  data() {
    return {
      query: "",
      suggestions: [],
      map: null,
      marker: null,
    };
  },
  mounted() {
    this.initializeMap();
  },
  methods: {
    fetchSuggestions() {
      if (this.query.length < 3) {
        this.suggestions = [];
        return;
      }
      axios
        .get("https://nominatim.openstreetmap.org/search", {
          params: {
            q: this.query,
            format: "json",
            addressdetails: 1,
            countrycodes: "VN",
          },
        })
        .then((response) => {
          this.suggestions = response.data;
        })
        .catch((error) => {
          console.error("Error fetching suggestions:", error);
        });
    },
    debouncedFetchSuggestions: debounce(function () {
      this.fetchSuggestions();
    }, 300),
    initializeMap() {
      this.map = L.map("map").setView([21.0285, 105.8542], 13); // Set initial view to Hanoi, Vietnam
      L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
        attribution: "© OpenStreetMap contributors",
      }).addTo(this.map);
    },
    selectSuggestion(suggestion) {
      const { lat, lon, display_name } = suggestion;
      if (this.marker) {
        this.map.removeLayer(this.marker);
      }
      this.marker = L.marker([lat, lon])
        .addTo(this.map)
        .bindPopup(display_name)
        .openPopup();
      this.map.setView([lat, lon], 13);
    },
  },
};
</script>

<style>
#map {
  height: 500px;
}
</style>
