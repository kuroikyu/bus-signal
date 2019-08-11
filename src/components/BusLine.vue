<template>
<div class="bus-container">
  <div class="line-number" v-bind:style="{ backgroundColor: line.lineColor, color: line.textColor }">{{ line.lineLabel }}</div>
  <div class="departure-time">{{ line.departure_time }}</div>
  <div class="time-to-departure">{{ line.departure_time | distanceInWordsToNow }}</div>
</div>
</template>

<script>
import dateFns from 'date-fns'

export default {
  props: {
    line: {
      type: Object,
      required: true
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
}
</script>

<style >
.bus-container {
  display: grid;
  grid-template-columns: 1fr 3fr;
  border-radius: 13px;
  border: 2px solid #2c3e50;
  text-align: center;
  vertical-align: middle;
}

li:first-child .bus-container {
  grid-template-columns: 1fr;
  grid-template-rows: 3fr 1fr 2fr;
  width: 8em;
  min-height: 170px;
  grid-row-end: span 4;
  justify-self: center;
  align-self: center;
}

li:first-child .bus-container > .line-number {
  border-radius: 10px 10px 0 0;
  font-size: 2.5em;
}

li:first-child .bus-container > .departure-time {
  display: flex;
  align-items: flex-end;
}

li:first-child .bus-container > .time-to-departure {
  padding: 0 5px;
}

.line-number,
.departure-time,
.time-to-departure {
  display: flex;
  justify-content: center;
  align-items: center;
}

.line-number {
  border-radius: 10px 0 0 10px;
  font-size: 1.5em;
  padding: 6px;
}

.departure-time {
  display:none;
}

.time-to-departure {
  padding: 0 5px;
  border-radius: 0 0 10px 10px;
}
</style>
