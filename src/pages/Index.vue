<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Search"
        dark
        borderless>
        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn
          @click="getWeatherBySearch"
            round
            dense
            flat
            icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name}}</div>
        <div class="text-h6 text-weight-light">{{ weatherData.weather[0].main }}</div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;F</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" alt="icon" />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Wuasar<br />Weather</div>
        <q-btn class="col" flat @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
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
      gettingLocation: false,
      apiKey: "f1e3293e05916b94fcb5d28b5c8eeeed",
      apiURL: "https://api.openweathermap.org/data/2.5/weather",
      imgUrl: "http://openweathermap.org/img/wn/"
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation() {
      this.$q.loading.show()

      if (this.$q.platform.is.electron || 
          this.$q.platform.is.chrome) {
        this.$axios.get('https://freegeoip.app/json/')
        .then(response => {
          this.lat = response.data.latitude
          this.lon = response.data.longitude
          this.getWeatherByCoords()
          this.$q.loading.hide()
        }) 
       .catch(error => {
          console.log("electron error getting coords")
          this.$q.loading.hide()
        })
      }
      else { 
      // browser based geoLocation
      // doesn't work in Chrome.  works for safari in https
      // in chrome, after the call, chrome location is disabled
      //     in system preferences!!
      console.log("positionCalled");
      if(!("geolocation" in navigator)) {
        console.log('Geolocation is not available.');
      }
      this.gettingLocation = true;
      navigator.geolocation.getCurrentPosition(function () {}, function () {}, {});
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.gettingLocation = false;
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          console.log(this.lat, this.lon)
          this.getWeatherByCoords()
          this.$q.loading.hide()
        },
        (err) => {
          this.gettingLocation = false;
          console.log("pos call failed");
          console.log(err.message);
          this.$q.loading.hide()
        },{
            enableHighAccuracy: true,
            timeout: 5000,
            maximumAge: 3000
        }
      );
      }

    },
    getWeatherByCoords() {
      this.$q.loading.show()
      this.$axios(`${this.apiURL}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=imperial`)
      .then(response => {
        this.weatherData = response.data
        //console.log(response)
        //console.log(this.weatherImg)
        this.$q.loading.hide()
      })
      .catch(error => {
        console.log("error getting coords")
        this.$q.loading.hide()
      })
    },
    getWeatherBySearch() {
      this.$q.loading.show()
      this.$axios(`${this.apiURL}?q=${this.search}&appid=${this.apiKey}&units=imperial`)
      .then(response => {
        this.weatherData = response.data
        //console.log(response)
        //console.log(this.weatherImg)
        this.$q.loading.hide()
      })
      .catch(error => {
        console.log("error getting by search")
        this.$q.loading.hide()
      })
    }
  },
};
</script>

<style lang="sass">
  .q-page
    background: linear-gradient(to top, #96c93d, #00b09b)
    &.bg-night
      background: linear-gradient(to top, #0f2027, #203a43)
    &.bg-day
      background: linear-gradient(to top, #2980b9, #6dd5fa)
      
  .degree
    top: -44px
  .skyline
    flex: 0 0 100px
    background: url(../../public/skyline.png)
    background-size: contain
    background-position: center bottom
</style>