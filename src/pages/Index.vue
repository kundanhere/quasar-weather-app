<template>
  <q-page class="flex column" :class="bgColor">
    <div class="col q-pt-lg q-px-md">
      <q-input
        @keyup.enter="getWeatherBySearch"
        v-model="search"
        placeholder="Search"
        color="white"
        borderless
        dark
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>

        <template v-slot:append>
          <q-icon
            v-if="search !== ''"
            name="close"
            @click="search = ''"
            class="cursor-pointer"
          />
          <q-icon name="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 text-weight-light relative-position degree"
            >&deg;C</span
          >
        </div>
        <div class="col text-center">
          <img
            :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          />
        </div>
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <wbr />
          Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="2.4em" name="my_location" />
          <div class="text-weight-light">Find my location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script>
export default {
  name: "PageIndex",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      lon: null,
      apiURL: "http://api.openweathermap.org/data/2.5/weather",
      apiKey: process.env.VUE_APP_API_KEY,
    };
  },
  computed: {
    bgColor() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("d")) {
          return "bg-day";
        } else {
          return "bg-night";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();

      if (this.$q.platform.is.electron) {
        this.$axios.get("https://freegeoip.app/json/").then((res) => {
          this.lat = res.data.latitude;
          this.lon = res.data.longitude;
          this.getWeatherByCoords();
          this.$q.loading.hide();
        });
      } else {
        navigator.geolocation.getCurrentPosition((pos) => {
          this.lat = pos.coords.latitude;
          this.lon = pos.coords.longitude;
          this.getWeatherByCoords();
          this.$q.loading.hide();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios
        .get(
          `${this.apiURL}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`
        )
        .then((res) => {
          this.weatherData = res.data;
          this.$q.loading.hide();
        });
    },
    getWeatherBySearch() {
      this.$q.loading.show();
      this.$axios
        .get(
          `${this.apiURL}?q=${this.search}&appid=${this.apiKey}&units=metric`
        )
        .then((res) => {
          this.weatherData = res.data;
          this.$q.loading.hide();
        });
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: #136a8a;  /* "fallback for old browsers" */
  background: -webkit-linear-gradient(to bottom, #267871, #136a8a);  /* "Chrome 10-25, Safari 5.1bg-blue-6" */
  background: linear-gradient(to bottom, #267871, #136a8a); /* "W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+" */
  &.bg-day
    background: linear-gradient(to bottom, #00b4db, #0083b0)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
.q-page .degree
  top: -44px
.q-page .skyline
  flex: 0 0 100px
  background: url(../assets/skyline.png)
  background-size: contain
  background-position: center bottom
</style>
