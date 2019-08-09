<template>
  <div id="app">
    <h1>Bus Service</h1>
    <section v-if="errored">
      <p>We're sorry, we're not able to retrieve this information at the moment, please try back later.</p>
    </section>
    <section v-else>
      <div
        v-for="(busData, stopName) in info"
        v-bind:key="stopName"
      >
        <h3>{{ stopName }}</h3>
        <ul>
          <li v-if="busData.length === 0">Loading...</li>
          <li
            v-else
            v-for="bus in busData"
            v-bind:key="bus.id"
          >
            <BusLine :line="bus"/>
          </li>
        </ul>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios'

import {APP_ID, APP_KEY} from '../api-details'
import {BUS_STOPS, KEY_LINES} from '../bus-details'

import BusLine from './components/BusLine'

export default {
  name: 'app',
  components: {
    BusLine
  },
  data() {
    return {
      info: null,
      errored: false
    }
  },
  mounted() {
    this.info = {}
    // Pre-fetch stop names so they're already sorted and sorting doesn't depend on the API response
    BUS_STOPS.forEach(busStop => {
      this.info = {...this.info, [busStop.name]: []}
    })

    BUS_STOPS.forEach(busStop => {
      axios
        .get(`https://transportapi.com/v3/uk/bus/stop/${busStop.atcoCode}/live.json?app_id=${APP_ID}&app_key=${APP_KEY}&group=route&nextbuses=no`)
        .then(response => response.data.departures)
        .then(departures => {
          // take only buses in the key list, remove not found values and flatten the resulting array
          const busService = [...KEY_LINES.map(line => departures[line.id])].filter(el => el).flat()

          // discard unneeded attributes and simplify names for the remaining ones
          const formatedBusService = busService.map((bus, i) => {
            // Grab details for the current line: colors, label, etc
            const [lineDetails] = KEY_LINES.filter(line => line.id === bus.line )

            // build final object structure
            return {
              id: `id${busStop.atcoCode}${i}`,
              line: bus.line,
              lineLabel: lineDetails.label,
              lineColor: lineDetails.color,
              direction: `${bus.direction} - ${bus.dir}`,
              departure_time: bus.best_departure_estimate,
            }
          })

          // Sort buses from shortest to longest arrival time
          const sortedBusService = formatedBusService.sort((a, b) => a.departure_time.localeCompare(b.departure_time))

          // finally, assign result to Vue's data object
          this.info[busStop.name] = sortedBusService

        })
      .catch(error => {
        console.error(error)
        this.errored = true
      })
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
</style>
