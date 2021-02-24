<template>
  <div class="text-white mb-8">
    <div class="places-input text-gray-800">
      <app-places
        :type="type"
        :api-key="apiKey"
        :app-id="appId"
        :key="`${appId}-${apiKey}`"
        @change="suggestion = $event.suggestion"
      />

    </div>
    <div class="weather-container font-sans w-128 max-w-lg overflow-hidden rounded-lg bg-gray-900 shadow-lg mt-4">
      <div class="current-weather flex items-center justify-between px-6 py-8">
        <div class="flex items-center">
          <div>
            <div class="text-6xl font-semibold">{{currentTemperature.actual}}℃</div>
            <div>Feels like {{currentTemperature.feels}}℃</div>
          </div>
          <div class="mx-5">
            <div class="font-semibold">{{currentTemperature.summary}}</div>
            <div>{{location.name}}</div>
          </div>
        </div>
        <div><img :src="'http://openweathermap.org/img/wn/' + currentTemperature.icon + '@2x.png'"
                  :alt="currentTemperature.summary"></div>
      </div>

      <div class="future-weather text-sm bg-gray-800 px-6 py-8 overflow-hidden">
        <div
          v-for="(day,index) in daily"
          :key="day.time"
          class="flex items-center"
          :class="{'mt-8' : index > 0}"
          v-if="index < 5"
        >
          <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(day.dt) }}</div>
          <div class="w-4/6 px-5 flex items-center">
            <div><img :src="'http://openweathermap.org/img/wn/' + day.weather[0].icon + '.png'"
                      :alt="day.weather[0].description"></div>
            <div class="ml-3">{{day.weather[0].description}}</div>
          </div>
          <div class="w-1/6 text-right">
            <div>{{Math.round(day.temp.max)}}℃</div>
            <div>{{Math.round(day.temp.min)}}℃</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import AppPlaces from './Places';

  export default {
    mounted() {
      this.fetchData()
    },
    components: { AppPlaces },
    data() {
      return {
        suggestion: undefined,
        appId: undefined,
        apiKey: undefined,
        type: 'city',
        currentTemperature: {
          actual: '',
          feels: '',
          summary: '',
          icon: '',
        },
        daily: [],
        location: {
          'name': 'Athens, Greece',
          'lat': 37.983810,
          'lng': 23.727539,
        }
      }
    },
    methods: {
      fetchData() {
        fetch(`/api/weather?lat=${this.location.lat}&lng=${this.location.lng}`)
          .then(response => response.json())
          .then(data => {
            this.currentTemperature.actual = Math.round(data.current.temp)
            this.currentTemperature.feels = Math.round(data.current.feels_like)
            this.currentTemperature.summary = data.current.weather.[0].description
            this.currentTemperature.icon = data.current.weather.[0].icon

            this.daily = data.daily
          })
      },
      toDayOfWeek(timestamp) {
        const newDate = new Date(timestamp*1000)
        const days = ['ΚΥΡ','ΔΕΥ','ΤΡΙ','ΤΕΤ','ΠΕΜ','ΠΑΡ','ΣΑΒ']

        return days[newDate.getDay()]
      },
      toggleApiKey() {
        if (this.apiKey) {
          this.apiKey = undefined;
          this.appId = undefined;
        } else {
          this.apiKey = 'bbb131a088fecf26387cea3ca399069b';
          this.appId = 'XCW6SGC3QO';
        }
      },
      toggleType() {
        if (this.type === 'city') {
          this.type = 'country';
        } else {
          this.type = 'city';
        }
      },
    },
    watch: {
      suggestion: {
        handler(suggestion, oldValue) {
          this.location.name = `${suggestion.name}, ${suggestion.country}`
          this.location.lat = suggestion.latlng.lat
          this.location.lng = suggestion.latlng.lng
          this.fetchData()
        },
        deep: true
      }
    }
  }
</script>
