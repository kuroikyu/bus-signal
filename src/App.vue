<template>
  <div id="app">
    <h1>Bus Service</h1>
    <section v-if="errored">
      <p>We're sorry, we're not able to retrieve this information at the moment, please try back later.</p>
    </section>
    <section v-else>
      <div v-if="loading">Loading...</div>
      <div
        v-else
        v-for="(busData, stopName) in info"
        v-bind:key="stopName"
      >
        <h3>{{ stopName }}</h3>
        <ul>
          <li
            v-for="bus in busData"
            v-bind:key="bus.id"
          >
            <span class="busLine">{{ bus.line }}: </span>
            <strong>{{ bus.departure_time }} </strong>
            <span class="departureTime"> {{ bus.departure_time | distanceInWordsToNow }}</span>
          </li>
        </ul>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios'
import dateFns from 'date-fns'

import {APP_ID, APP_KEY} from '../api-details'
import {BUS_STOPS, KEY_LINES} from '../bus-details'

export default {
  name: 'app',
  data() {
    return {
      info: null,
      loading: true,
      errored: false
    }
  },
  filters: {
    distanceInWordsToNow (time) {
      const [hours, minutes] = time.split(':')
      const departureDateTime = new Date()
      departureDateTime.setHours(hours)
      departureDateTime.setMinutes(minutes)
      departureDateTime.setSeconds('59')
      departureDateTime.setMilliseconds('999')


      const distanceToNow = dateFns.distanceInWordsToNow(
          departureDateTime,
          {addSuffix: true, includeSeconds:true}
      )
      return distanceToNow
    }
  },
  mounted() {
    this.info = {}
    BUS_STOPS.forEach(busStop => {
      axios
        .get(`https://transportapi.com/v3/uk/bus/stop/${busStop.atcoCode}/live.json?app_id=${APP_ID}&app_key=${APP_KEY}&group=route&nextbuses=no`)
        .then(response => response.data.departures)
        .then(departures => {
          // take only buses in the key list, remove not found values and flatten the resulting array
          const busService = [...KEY_LINES.map(line => departures[line])].filter(el => el).flat()
          // discard unneeded attributes and simplify names for the remaining ones
          const neatBusService = busService.map((bus, i) => {
            return {
              id: `id${busStop.atcoCode}${i}`,
              line: bus.line,
              direction: `${bus.direction} - ${bus.dir}`,
              departure_time: bus.best_departure_estimate,
            }
          })
          // assign result to vue's data object
          this.info = {...this.info, [busStop.name]: neatBusService}

        })
      .catch(error => {
        console.error(error)
        this.errored = true
      })
      .finally(() => this.loading = false)
    })
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
.busLine {
}
.departureTime {
  color: #1aad63;
  color: #701dc3;
}
</style>
