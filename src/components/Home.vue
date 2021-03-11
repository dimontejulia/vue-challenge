<script>
import api from "./../apiBridge";
import router from "./../router";
import ButtonGroup from "./vuestrap/buttonGroup.vue";
import Radio from "./vuestrap/Radio.vue";
import axios from "axios";

export default {
  name: "home",

  components: {
    radio: Radio,
    buttonGroup: ButtonGroup,
  },

  created: function () {
    // private data
  },

  methods: {
    getStoreList() {
      api.calls.getStoreList(null, (stores, errTxt, errCode) => {
        if (errTxt) {
          this.error = errTxt;
        } else {
          this.storeList = stores;
        }
      });
    },

    storeClicked(store) {
      console.log("Store clicked:  " + store.storeNumber, store.city);
      // Make store details api call here
      this.currentStore = store;
      this.getForecast();
      this.mapUrl = `https://www.openstreetmap.org/export/embed.html?bbox=${this.currentStore.longitude}%2C${this.currentStore.latitude}%2C${this.currentStore.longitude}%2C${this.currentStore.latitude}&amp;layer=mapnik&amp;marker=${this.currentStore.latitude}%2C${this.currentStore.longitude}`;
      console.log(this.mapUrl);
      console.log("current store: ", this.currentStore);
    },

    getForecast() {
      console.log("fetching weather data for: ", this.currentStore);
      const BASE_URL = "http://api.openweathermap.org/data/2.5/weather?";
      const API_KEY = "0baac89f1d0c315bcb40c54c80beac3c";
      const URL = `${BASE_URL}lat=${this.currentStore.latitude}&lon=${this.currentStore.longitude}&appid=${API_KEY}`;
      axios
        .get(URL)
        .then((response) => (this.weatherForecast = response.data))
        .catch((err) => console.log(err));
    },
  },

  mounted: function () {
    console.log("Main page");
    this.getStoreList();
  },

  data() {
    return {
      viewOption: "L",
      storeList: [],
      currentStore: {},
      error: "",
      weatherForecast: {},
      mapUrl: ``,
    };
  },
};
</script>

<template>
  <div>
    <div v-if="error" class="alert alert-danger">
      <p><span v-html="error"></span></p>
    </div>

    <button-group style="display: flex; margin: auto">
      <radio
        v-model="viewOption"
        checked-value="L"
        @keyup.native.enter="viewOption = 'L'"
        tabindex="350"
        >List View</radio
      >
      <radio
        v-model="viewOption"
        checked-value="M"
        @keyup.native.enter="viewOption = 'M'"
        tabindex="351"
        >Map View</radio
      >

      <radio
        v-model="viewOption"
        checked-value="W"
        @keyup.native.enter="viewOption = 'W'"
        tabindex="351"
        >Get Weather</radio
      >
    </button-group>

    <div v-if="viewOption == 'L' && storeList.length">
      <div v-for="(store, index) in storeList" v-bind:key="index">
        <div
          @click="storeClicked(store)"
          style="
            display: table;
            width: 100%;
            margin-left: 10px;
            margin-top: 15px;
            cursor: pointer;
            color: white;
          "
        >
          <span style="display: table-cell; width: 2em; vertical-align: middle">
            <input
              type="radio"
              name="pickupChoice"
              :value="store.storeNumber"
              :checked="
                currentStore && store.storeNumber == currentStore.storeNumber
              "
              class="option-input radio"
            />
          </span>

          <div v-if="store.details && store.details.street">
            <b
              ><div>
                {{ store.details.streetNumber }} {{ store.details.street }}
                <span v-if="store.primary" style="font-weight: normal"
                  >&nbsp;&nbsp;{{ $t("primary") }}</span
                >
              </div>
              <div>
                {{ store.details.city }} {{ store.details.province }}
                {{ store.details.postalCode }}
              </div></b
            >
            <div class="hours">{{ store.details.closingTimeMsg }}</div>
          </div>
          <div v-else>
            <b
              ><div>{{ formatRawStoreName(store.storeName) }}</div></b
            >
          </div>
        </div>
        <hr
          v-if="index < storeList.length - 1"
          style="margin: 10px 0px 0px 0px; max-width: 100%"
        />
      </div>
    </div>

    <div v-if="viewOption == 'W' && storeList.length" style="color: white">
      <div v-if="Object.keys(currentStore).length > 0">
        <h2>Location: {{ currentStore.city }}, {{ currentStore.province }}</h2>
        <p>
          {{ currentStore.details.streetNumber }}
          {{ currentStore.details.street }} {{ currentStore.details.city }}
          {{ currentStore.details.province }}
          {{ currentStore.details.postalCode }}
        </p>

        <!-- temperature -->
        <!-- weather description -->
        <h4>Current Weather:</h4>
        <h4>
          {{ weatherForecast.weather[0].main }}:
          {{ weatherForecast.weather[0].description }}
        </h4>
        <h4>Temperature:</h4>
        <ul>
          <li>Temperature: {{ weatherForecast.main.temp }}</li>
          <li>Feels Like: {{ weatherForecast.main.feels_like }}</li>
          <li>High Of: {{ weatherForecast.main.temp_max }}</li>
          <li>Low Of: {{ weatherForecast.main.temp_min }}</li>
          <li>Humidity: {{ weatherForecast.main.humidity }}%</li>
        </ul>
      </div>
      <div v-if="Object.keys(currentStore).length === 0">
        <h3>No location selected.</h3>
      </div>
    </div>

    <div v-if="viewOption == 'M' && storeList.length" style="color: white">
      <h1>Map</h1>
      <div v-if="Object.keys(currentStore).length > 0">
        <iframe
          :src="mapUrl"
          width="1000"
          height="600"
          frameborder="0"
          scrolling="no"
          marginheight="0"
          marginwidth="0"
          style="border: 2px solid white"
        ></iframe
        ><br /><small><a :href="mapUrl">View Larger Map</a></small>
      </div>
      <div v-if="Object.keys(currentStore).length === 0">
        <h3>No location selected.</h3>
      </div>
    </div>
  </div>
</template>
