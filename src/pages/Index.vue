<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
      <div class="col text-white text-weight-thin">{{ todaysDate }}</div>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h3 relative-position degree">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <br />
          Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col silhouette"></div>
  </q-page>
</template>

<script>
import { date } from "quasar";
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: "http://api.openweathermap.org/data/2.5/weather",
      apiKey: "c8713f3b29c758b62d16bc6e701e76dc",
    };
  },
  computed: {
    todaysDate() {
      let timeStamp = Date.now();
      return date.formatDate(timeStamp, "dddd DD MMMM");
    },
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();
      if (this.$q.platform.electron) {
        this.$axios.get(`https://freegeoip.app/json/`).then(response => {
          this.lat=response.data.latitude
          this.lon=response.data.longitude
          this.getWeatherByCoords()
        })
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position: ", position);
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },

    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios(
        `http://api.openweathermap.org/data/2.5/weather?q=${this.search}&appid=c8713f3b29c758b62d16bc6e701e76dc&units=metric`
      ).then((response) => {
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to top, #24c6dc, #514a9d)
  &.bg-night
    background: linear-gradient(to top, #bdc3c7, #2c3e50)
  &.bg-day
    background: linear-gradient(to bottom, #2980b9, #6dd5fa, #ffffff)
.degree
  top: -35px
.silhouette
  flex: 0 0 100px
  background: url(../assets/silhouette.png)
  background-size: contain
  background-position: center bottom
</style>
