<template>
  <el-container style="height: 100vh">
    <StartAndEndForm>
      <el-input v-model="titleInput" placeholder="請輸入標題" />
      <el-switch
        v-model="startEndType"
        active-text="經緯度"
        inactive-text="輸入地點"
      />
      <div v-if="startEndType">
        <span style="color: black">起點：</span>
        <el-input v-model="start.lat" placeholder="起點經度" />
        <el-input v-model="start.lng" placeholder="起點緯度" />

        <span style="color: black">終點：</span>
        <el-input v-model="end.lat" placeholder="終點經度" />
        <el-input v-model="end.lng" placeholder="終點緯度" />
      </div>
      <div v-if="!startEndType">
        <span style="color: black">起點地址：<br />{{ start.address }}</span>
        <AutoCompletePlace placeholder="輸入起點" @select="selectStart" />
        <span style="color: black">終點地址：<br />{{ end.address }}</span>
        <AutoCompletePlace placeholder="輸入終點" @select="selectEnd" />
      </div>
      <el-switch
        v-model="inputType"
        active-text="手動選擇"
        inactive-text="輸入文字"
      />
      <div v-if="inputType">
        <el-radio-group v-model="mode" style="margin-bottom: 0.5vh">
          <el-radio label="DRIVING">開車</el-radio>
          <el-radio label="WALKING">步行</el-radio>
          <el-radio label="BICYCLING">騎車</el-radio>
        </el-radio-group>
        <el-radio-group v-model="splitRange">
          <el-radio label="2000">小</el-radio>
          <el-radio label="10000">大</el-radio>
          <el-radio label="9999999">不切</el-radio>
        </el-radio-group>
      </div>
      <div v-if="!inputType">
        <el-input
          type="textarea"
          :rows="3"
          placeholder="輸入偏好資訊"
          v-model="note"
        />
      </div>
      <el-button type="primary" @click="getPointsInDirection" style="">
        取得地點
      </el-button>
      <el-button @click="segPrev">-</el-button>
      <span style="color: black; margin-left: 5px; margin-right: 5px">
        {{ seg_count + 1 }}/{{ allData.nearbys.length }}
      </span>
      <el-button @click="segNext"> + </el-button>
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
      <el-button @click="recommandPrev">prev</el-button>
      <el-card
        v-for="(marker, index) in filtedMarkerList"
        style="width: 16vw"
        :key="index"
        :body-style="{ padding: '0px' }"
        @mouseover="hoverMarker(index)"
      >
        <div style="padding: 14px">
          <el-link
            :href="
              'https://www.google.com/maps/place/?q=place_id:' + marker.place_id
            "
            target="_blank"
            >{{ marker.name.slice(0, 20) }}
          </el-link>
          <div class="bottom">
            評分: {{ marker.rating }} ({{ marker.user_ratings_total }})
            (相似：{{ Math.round(marker.similarity * 100) / 100 }}) <br />
          </div>
        </div>
      </el-card>
      <el-button @click="recommandNext">next</el-button>
    </el-space>
    <l-map
      ref="rMap"
      style="width: 100%; height: 100%"
      :zoom="zoom"
      :center="center"
      :maxZoom="18"
    >
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <l-marker :lat-lng="start" :key="start.address">
        <l-popup> start: {{ start.lat }} {{ start.lng }} </l-popup>
      </l-marker>
      <l-marker :lat-lng="end" :key="end.address">
        <l-popup> end: {{ end.lat }} {{ end.lng }} </l-popup>
      </l-marker>

      <l-marker
        ref="rMarker"
        v-for="(marker, index) in filtedMarkerList"
        :key="index"
        :lat-lng="marker.geometry.coordinates"
        :title="marker.place_id"
        @mouseover="hoverMarker(index)"
      >
        <l-icon
          :icon-size="[18, 24]"
          :icon-url="marker.icon"
          class-name="custom-div-icon"
        />
        <l-popup>
          <el-link
            :href="`https://www.google.com/maps/place/?q=place_id:${marker.place_id}`"
            target="_blank"
          >
            名稱: {{ marker.name }}
          </el-link>
          <br />
          星數: {{ marker.rating }} <br />
          評論數: {{ marker.user_ratings_total }} <br />
          標籤：
          <div v-if="marker.tags">
            <el-tag v-for="tag in marker.tags.slice(0, 5)" :key="tag">{{
              tag
            }}</el-tag>
          </div>
          <a
            v-if="marker.tags.length > 5 && !expandTags"
            @click="expandTags = !expandTags"
            style="cursor: pointer"
            >(...)</a
          >
          <div v-if="expandTags">
            <el-tag v-for="tag in marker.tags.slice(5, -1)" :key="tag">
              {{ tag }}
            </el-tag>
            <a @click="expandTags = !expandTags" style="cursor: pointer"
              >收起</a
            >
          </div>
        </l-popup>
      </l-marker>
      <l-circle
        v-for="(path, index) in allData.paths"
        :key="index"
        :lat-lng="path[0]"
        :radius="30"
        color="#3388ff"
        :weight="3"
        fill
        fill-color="#ffffff"
        :fill-opacity="1"
      />
      <l-polyline
        class-name="interactive-polyline"
        v-for="(path, index) in allData.paths"
        :key="index"
        :lat-lngs="path"
        :weight="4"
        @click="clickPolyline(index)"
      ></l-polyline>
      <l-polyline :lat-lngs="polyline.latLngs" :weight="8" color="#c6e2ff" />
    </l-map>
  </el-container>
</template>

<script setup>
import "leaflet/dist/leaflet.css";
import {
  LMap,
  LTileLayer,
  LMarker,
  LCircle,
  LIcon,
  LPopup,
  LPolyline,
} from "@vue-leaflet/vue-leaflet";
import axios from "axios";
import { ElLoading } from "element-plus";
import StartAndEndForm from "../components/StartAndEndForm.vue";
import AutoCompletePlace from "../components/AutoCompletePlace.vue";
import { ref, reactive, computed } from "vue";

const titleInput = ref("");

const url = "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png";
const attribution =
  '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors';
const zoom = ref(14);
const center = reactive({ lat: 25.033, lng: 121.565 });
const start = reactive({
  lat: 24.9885267,
  lng: 121.5739302,
  address: "（尚未選擇地點）",
});
const end = reactive({
  lat: 25.0342531,
  lng: 121.5270531,
  address: "（尚未選擇地點）",
});

const expandTags = ref(false);

const startEndType = ref(false);
const inputType = ref(false);
const mode = ref("DRIVING");
const splitRange = ref("10000");
const note = ref("");

const seg_count = ref(0);
const recommand_count = ref(-1);
const allData = reactive({
  nearbys: [],
  paths: [],
});

const selectStart = (place) => {
  console.log("change start", place);
  const url = `${import.meta.env.VITE_API_URL}/autoComplete/detail?place_id=${
    place.place_id
  }`;
  axios.get(url).then((res) => {
    start.lat = res.data.lat;
    start.lng = res.data.lng;
    start.address = res.data.address;
  });
};
const selectEnd = (place) => {
  console.log("change end", place);
  const url = `${import.meta.env.VITE_API_URL}/autoComplete/detail?place_id=${
    place.place_id
  }`;
  axios.get(url).then((res) => {
    end.lat = res.data.lat;
    end.lng = res.data.lng;
    end.address = res.data.address;
  });
};

const segPrev = () => {
  if (seg_count.value > 0) seg_count.value--;
  recommand_count.value = 0;
  clickPolyline(seg_count.value);
};

const segNext = () => {
  if (seg_count.value < allData.nearbys.length - 1) seg_count.value++;
  recommand_count.value = 0;
  clickPolyline(seg_count.value);
};

const clickPolyline = (index) => {
  const start = allData.paths[index][0];
  const end = allData.paths[index][allData.paths[index].length - 1];
  updateCenter(start, end);

  seg_count.value = index;
  recommand_count.value = 0;
};

const rMap = ref(null);

const updateCenter = (start, end) => {
  rMap.value.leafletObject.setView(
    [(start.lat + end.lat) / 2, (start.lng + end.lng) / 2],
    14
  );
};

const updateBounds = (start, end) => {
  rMap.value.leafletObject.fitBounds(
    [
      [start.lat, start.lng],
      [end.lat, end.lng],
    ],
    {
      padding: [50, 50],
    }
  );
};

const recommandPrev = () => {
  recommand_count.value > 0 ? (recommand_count.value -= 1) : 0;
};

const recommandNext = () => {
  recommand_count.value += 1;
  console.log(recommand_count.value);
};

const rMarker = ref(null);
const hoverMarker = (index) => {
  rMarker.value[index].leafletObject.openPopup();
};

const hoverMarkerOut = (index) => {
  rMarker.value[index].leafletObject.closePopup();
};

const fetchData = async () => {
  ElLoading.service({
    lock: true,
    text: "Loading",
    background: "rgba(0, 0, 0, 0.7)",
  });

  const queryVectors = async () => {
    const res = await axios.get(`${import.meta.env.VITE_PYAPI_URL}/word2Vec`, {
      params: {
        inputText: note.value,
      },
    });
    console.log("note: ", note.value, "queryVectors: ", res.data);
    return res.data.vectors;
  };

  const body = {
    originLat: start.lat,
    originLng: start.lng,
    destLat: end.lat,
    destLng: end.lng,
    limitDistance: 1,
    splitRange: parseInt(splitRange.value),
    queryString: note.value,
    queryVectors: await queryVectors(),
  };
  console.log("fetching...", body);
  const startTime = Date.now();
  const res = await axios.post(
    `${import.meta.env.VITE_API_URL}/direction/nearbyPoints`,
    body
  );
  console.log((res + "").length);
  console.log("fetching time: ", (Date.now() - startTime) / 1000, "s");
  seg_count.value = 0;
  recommand_count.value = 0;
  ElLoading.service().close();

  return res.data;
};

const getPointsInDirection = async () => {
  const before = Date.now();
  const resp = await fetchData();
  console.log("fetching done", resp);
  console.log("fetching time: ", (Date.now() - before) / 1000, "s");

  allData.nearbys = resp.nearbys;
  allData.paths = resp.paths;
  console.log("allData", allData);

  recommand_count.value = 0;

  const size = allData.paths.length;
  const start = allData.paths[0][0];
  const end = allData.paths[size - 1][allData.paths[size - 1].length - 1];
  updateBounds(start, end);
};

const polyline = computed(() => {
  console.log(
    "seg:",
    seg_count.value,
    " polyline:",
    allData.paths[seg_count.value]
  );
  return {
    latLngs: allData.paths[seg_count.value],
  };
});

const marker_list = computed(() => {
  var _marker_list = [];
  console.log("marker_list: ", allData.nearbys);
  for (var i = 0; i < allData.nearbys.length; i++) {
    const tmp = allData.nearbys[i].map((item) => {
      return {
        // reverse lat lng cause of mongodb is lng lat
        place_id: item.place_id,
        geometry: {
          coordinates: [
            item.geometry.coordinates[1],
            item.geometry.coordinates[0],
          ],
        },
        icon: item.icon,
        name: item.name,
        address: item.address,
        rating: item.rating,
        user_ratings_total: item.user_ratings_total,
        tags: item.tags,
        similarity: item.similarity,
      };
    });
    tmp.sort((a, b) => {
      if (a.similarity - b.similarity > 0.05) return -1;
      if (b.similarity - a.similarity > 0.05) return 1;

      var x = 0;
      if (a.rating || a.user_ratings_total) {
        x = a.rating * Math.sqrt(a.user_ratings_total);
      }
      var y = 0;
      if (b.rating || b.user_ratings_total) {
        y = b.rating * Math.sqrt(b.user_ratings_total);
      }
      if (x > y) {
        return -1;
      } else if (x < y) {
        return 1;
      } else {
        return 0;
      }
    });

    _marker_list.push(tmp);
  }
  console.log("_marker_list: ", _marker_list);
  return _marker_list;
});

const filtedMarkerList = computed(() => {
  const _tmp = marker_list.value[seg_count.value];
  console.log(_tmp);
  if (_tmp) {
    return _tmp.slice(recommand_count.value * 5, recommand_count.value * 5 + 5);
  } else {
    console.log("_tmp not exist");
    return [];
  }
});
</script>
<style scoped>
.image {
  width: 100%;
  display: block;
}
</style>
