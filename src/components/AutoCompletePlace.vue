<template>
  <el-autocomplete
    v-model="state"
    :fetch-suggestions="querySearch"
    :trigger-on-focus="false"
    clearable
    :placeholder="props.placeholder"
    @select="handleSelect"
    style="width: 100%; margin-bottom: 2vh"
  />
</template>
<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";

const props = defineProps({
  placeholder: {
    type: String,
    default: "",
  },
});

const emit = defineEmits(["select"]);

const state = ref("");

const restaurants = ref([]);
const querySearch = (queryString, cb) => {
  const url = "http://localhost:8888/autoComplete/predict?query=" + queryString;
  axios.get(url).then((res) => {
    restaurants.value = res.data;
    console.log(restaurants.value);
    cb(restaurants.value);
    // const results = queryString
    //   ? restaurants.value.filter(createFilter(queryString))
    //   : restaurants.value;
    // // call callback function to return suggestions
    // cb(results);
  });
};
const createFilter = (queryString) => {
  return (restaurant) => {
    return (
      restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
    );
  };
};
const loadAll = () => {
  return [
    {
      value: "政治大學",
      lat: 24.9885267,
      lng: 121.5739302,
      address: "116台北市文山區指南路二段64號",
      placeId: "123123123",
    },
    {
      value: "台灣大學",
      lat: 25.0342531,
      lng: 121.5270531,
      address: "10617台北市大安區羅斯福路四段1號",
      placeId: "1asdasd",
    },
    {
      value: "台灣師範大學",
      lat: 25.010756355323792,
      lng: 121.5399450752843,
      address: "106台北市大安區和平東路一段162號",
      placeId: "1asdasd",
    },
  ];
};

const loadAutoComplete = (str) => {};

const handleSelect = (place) => {
  console.log(place);
  emit("select", place);
};

onMounted(() => {
  restaurants.value = loadAll();
});
</script>
