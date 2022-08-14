<template>
  <l-map
    style="height: 100%"
    :zoom="zoom"
    :center="center"
    @update:center="centerUpdated"
  >
    <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
    <l-marker
      v-for="(marker, index) in marker_list"
      :key="index"
      :lat-lng="marker.location"
      :title="marker.name"
    >
      <l-popup>
        店名：{{ marker.name }} ｜ 地址：{{ marker.vicinity }}
      </l-popup>
    </l-marker>
  </l-map>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LMarker, LPopup } from "@vue-leaflet/vue-leaflet";
import newTaipeiRestaurants from "./newTaipeiRestaurants.json";

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup
  },
  props: {
    location: Array, // [lat, lng]
    trigger: Number,
  },
  data() {
    return {
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 16,
      // map_data: new_taipei,
      map_data: newTaipeiRestaurants,
      marker_list: [],
      center: this.location, // [lat, lng]
    };
  },
  // computed: {
  //   center() {
  //     return this.location;
  //   },
  // },
  watch: {
    location: function () {
      console.log(this.location);
    },
    trigger: function () {
      console.log("rec");
      this.onClick(parseFloat(this.center[0]), parseFloat(this.center[1]));
    },
  },
  methods: {
    centerUpdated(center) {
      console.log(center);
      this.center = [center.lat, center.lng];
    },
    distance(lat1, lon1, lat2, lon2, unit) {
      if (lat1 == lat2 && lon1 == lon2) {
        return 0;
      } else {
        var radlat1 = (Math.PI * lat1) / 180;
        var radlat2 = (Math.PI * lat2) / 180;
        var theta = lon1 - lon2;
        var radtheta = (Math.PI * theta) / 180;
        var dist =
          Math.sin(radlat1) * Math.sin(radlat2) +
          Math.cos(radlat1) * Math.cos(radlat2) * Math.cos(radtheta);
        if (dist > 1) {
          dist = 1;
        }
        dist = Math.acos(dist);
        dist = (dist * 180) / Math.PI;
        dist = dist * 60 * 1.1515;
        if (unit == "K") {
          dist = dist * 1.609344;
        }
        if (unit == "N") {
          dist = dist * 0.8684;
        }
        return dist;
      }
    },
    onClick(input_lat, input_lng) {
      console.log(input_lat, input_lng);
      this.marker_list = [];
      for (var i = 0; i < this.map_data.length; i++) {
        let lat = this.map_data[i].geometry.location.lat;
        let lng = this.map_data[i].geometry.location.lng;
        if (this.distance(input_lat, input_lng, lat, lng, "K") <= 0.2) {
          this.marker_list.push({
            name: this.map_data[i].name,
            location: [lat, lng],
            place_id: this.map_data[i].place_id,
            vicinity: this.map_data[i].vicinity,
          });
        }
      }

      console.log(this.marker_list);
    },
  },
  mounted() {
    console.log(this.map_data[0]);
  },
};
</script>
