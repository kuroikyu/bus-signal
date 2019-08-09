<template>
<div>
  <span>{{ line.lineLabel }}: </span>
  <strong>{{ line.departure_time }} </strong>
  <span v-bind:style="{ color: line.lineColor }"> {{ line.departure_time | distanceInWordsToNow }}</span>
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

<style scoped>
</style>
