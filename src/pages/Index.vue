<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        @keyup.enter="getWeatherBySearch"
        bottom-slots
        v-model="search"
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
    </div>

    <template v-if="weather">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weather.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weather.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weather.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="'http://openweathermap.org/img/wn/' + weatherImg + '.png'"
          alt
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Fleno's
          <br />Weather
        </div>
        <q-btn @click="getLocation" class="col" flat>
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
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weather: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather?',
      apiKey: 'f07b42ff2f34bb6767b4dcf8e2730dbb',
      weatherImg: '',
    }
  },
  computed: {
    bgClass() {
      if (this.weather) {
        if (this.weather.weather[0].icon.endsWith('n')) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show()

      // Electron-apps can't use navigator.geolocation
      // We will use freegeoip.app as fallback for electron to get an approx location
      if (this.$q.platform.is.electron) {
        this.$axios.get('https://freegeoip.app/json/').then(res => {
          this.lat = res.data.latitude
          this.lon = res.data.longitude
          this.getWeatherByCoords()
        })
      } else {
        navigator.geolocation.getCurrentPosition(position => {
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
        })
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show()
      let url =
        this.apiUrl +
        'lat=' +
        this.lat +
        '&lon=' +
        this.lon +
        '&apikey=' +
        this.apiKey +
        '&units=metric'
      console.log(url)
      this.$axios(url).then(res => {
        this.weather = res.data
        this.weatherImg = res.data.weather[0].icon
        this.$q.loading.hide()
      })
    },
    getWeatherBySearch() {
      this.$q.loading.show()
      let url =
        this.apiUrl +
        'q=' +
        this.search +
        '&apikey=' +
        this.apiKey +
        '&units=metric'
      console.log(url)
      this.$axios(url).then(res => {
        this.weather = res.data
        this.weatherImg = res.data.weather[0].icon
        this.$q.loading.hide()
      })
    },
  },
}
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-day
    background: linear-gradient(to bottom, #136a8a, #267871)
  &.bg-night
    background: linear-gradient(to bottom, #292e49,#536976)
.degree
  top: -44px
.skyline
  flex: 0 0 100px
  background: url(../statics/skyline.png)
  background-size: contain
  background-position: center bottom
  background-repeat: no-repeat
</style>
