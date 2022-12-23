<template>
  <el-container style="height: 100vh">
    <StartAndEndForm>
      <AutoCompletePlace
        :input="start.name"
        @change="start = $event"
        placeholder="輸入起點"
      />
      <AutoCompletePlace
        :input="end.name"
        @change="end = $event"
        placeholder="輸入終點"
      />
      <el-button type="primary" @click="getPointsInDirection" style="">
        取得地點
      </el-button>
    </StartAndEndForm>
    <el-space
      wrap
      :size="10"
      style="
        width: 98vw;
        position: absolute;
        z-index: 800;
        bottom: 1vh;
        left: 1vw;
      "
    >
      <el-button @click="recommand_count > 0 ? (recommand_count -= 5) : 0"
        >prev</el-button
      >
      <el-card
        v-for="(marker, index) in filted_marker_list"
        style="width: 16vw"
        :key="index"
        :body-style="{ padding: '0px' }"
      >
        <div style="padding: 14px">
          <el-link
            :href="
              'https://www.google.com/maps/place/?q=place_id:' + marker.place_id
            "
            >{{ marker.name.slice(0, 20) }}
          </el-link>
          <div class="bottom">
            評分: {{ marker.rating }} ({{ marker.user_ratings_total }}) <br />
          </div>
        </div>
      </el-card>
      <el-button @click="recommand_count += 5">next</el-button>
    </el-space>
    <l-map
      style="width: 100%; height: 100%"
      :zoom="zoom"
      :center="center"
      :maxZoom="18"
      @update:center="centerUpdated"
    >
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker :lat-lng="start" :key="start">
        <l-popup> {{ start.lat }} {{ start.lng }} </l-popup>
      </l-marker>
      <l-marker :lat-lng="end" :key="end">
        <l-popup> {{ end.lat }} {{ end.lng }} </l-popup>
      </l-marker>
      <l-marker
        v-for="(marker, index) in filted_marker_list"
        :key="index"
        :lat-lng="marker.geometry.coordinates"
        :title="marker.place_id"
      >
        <l-icon
          :icon-size="[18, 24]"
          :icon-url="marker.icon"
          class-name="custom-div-icon"
        />
        <l-popup>
          name: {{ marker.name }} <br />
          rating: {{ marker.rating }} <br />
          user_ratings_total: {{ marker.user_ratings_total }} <br />
          <!-- place_id: {{ marker.place_id }} <br />
          icon: {{ marker.icon }} -->
        </l-popup>
      </l-marker>

      <l-polyline
        :lat-lngs="polyline.latLngs"
        :color="polyline.color"
        :weight="6"
      ></l-polyline>
    </l-map>
  </el-container>
</template>

<script>
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LMarker,
  LIcon,
  LPopup,
  LPolyline,
} from "@vue-leaflet/vue-leaflet";
import axios from "axios";
import { ElLoading } from "element-plus";
import StartAndEndForm from "../components/StartAndEndForm.vue";
import AutoCompletePlace from "../components/AutoCompletePlace.vue";

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LIcon,
    LPopup,
    LPolyline,
    StartAndEndForm,
    AutoCompletePlace,
  },
  data() {
    return {
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 14,
      marker_list: [],
      polyline: {
        latLngs: [],
        color: "green",
      },
      center: { lat: 25.010756355323792, lng: 121.5399450752843 },
      start: {
        lat: 24.9885267,
        lng: 121.5739302,
        name: "台北車站",
      },
      end: {
        lat: 25.0342531,
        lng: 121.5270531,
        name: "台北101",
      },
      recommand_count: -1,
      filted_marker_list: [],
    };
  },
  watch: {
    recommand_count: function (newVal, oldVal) {
      console.log(newVal, oldVal);
      // const sorted = this.marker_list;
      // this.filted_marker_list = [];
      // for (let i = newVal; i < newVal + 10; i++) {
      //   this.filted_marker_list.push(sorted[i]);
      // }
      // console.log(sorted);
      this.filted_marker_list = this.marker_list.slice(newVal, newVal + 5);
    },
  },
  methods: {
    centerUpdated(center) {
      console.log(center);
      this.center = { lat: center.lat, lng: center.lng };
    },
    getPointsInDirection() {
      const before = Date.now();
      const fetchData = async () => {
        ElLoading.service({
          lock: true,
          text: "Loading",
          background: "rgba(0, 0, 0, 0.7)",
        });
        const params = {
          originLat: this.start.lat,
          originLng: this.start.lng,
          destLat: this.end.lat,
          destLng: this.end.lng,
          limitDistance: 0.2,
        };
        console.log("fetching...", params);
        const res = await axios.get(
          "http://localhost:8888/direction/nearbyPoints",
          {
            params: params,
          }
        );
        console.log("response: ", JSON.stringify(res.data));
        this.marker_list = res.data.nearby.map((item) => {
          return {
            place_id: item.place_id,
            name: item.name,
            rating: item.rating,
            user_ratings_total: item.user_ratings_total,
            geometry: {
              coordinates: [
                item.geometry.coordinates[1],
                item.geometry.coordinates[0],
              ],
            },
            icon: item.icon,
          };
        });
        this.marker_list.sort((a, b) => {
          var x = 0;
          if (a.rating || a.user_ratings_total) {
            x = a.rating * a.user_ratings_total;
          }
          var y = 0;
          if (b.rating || b.user_ratings_total) {
            y = b.rating * b.user_ratings_total;
          }
          if (x > y) {
            return -1;
          } else if (x < y) {
            return 1;
          } else {
            return 0;
          }
        });
        console.log(this.marker_list);
        this.polyline.latLngs = res.data.path;
        this.recommand_count = 0;
        ElLoading.service().close();
        console.log("fetching time: ", (Date.now() - before) / 1000, "s");
      };
      fetchData();
    },
  },
  mounted() {},
  computed: {},
};
</script>
<style scoped>
.image {
  width: 100%;
  display: block;
}
</style>
