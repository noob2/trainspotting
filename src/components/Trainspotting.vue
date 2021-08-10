<template>
  <div style="margin: 30px auto; width: 90%;font-weight:bold">
    <div v-if="innerWidth >= 1024" style="justify-content: space-between; display: flex;">
      <span>
        Jiminny Trainspotting
        <button v-if="!interval" @click="start">Start</button>
        <button v-else @click="stop">Stop</button>
      </span>
      <span>
        Trains in transit: {{ trainsInTransit.length ? trainsInTransit.toString() : 'None' }} &nbsp;&nbsp;&nbsp;
        <span v-if="currentTime">Current Time: {{ formatDatehhmm(currentTime) }}</span>
      </span>
    </div>
    <div v-else>
      Jiminny Trainspotting
    </div>
    <table>
      <tr>
        <th v-if="innerWidth >= 1300 || innerWidth < 1024">Name</th>
        <th v-if="innerWidth >= 1300 || innerWidth < 1024">Route</th>
        <th v-else>Name / Route</th>
        <th>Timetable</th>
        <th v-if="innerWidth >= 1024">Next Station</th>
        <th>Train</th>
      </tr>
      <tr v-for="journey in journeys" :key="journey.name">
        <td v-if="innerWidth >= 1300 || innerWidth < 1024">{{ journey.name }}</td>
        <td v-if="innerWidth >= 1300 || innerWidth < 1024">{{ journey.route }}</td>
        <td v-else>{{ journey.name + ' / ' + journey.route }}</td>
        <td>
          <div v-if="innerWidth >= 1024">
            <div class="timetable">
              <div
                v-for="timetable in journey.timetable"
                :key="timetable.station"
                style="display:inline-block"
                :style="`margin-left:${timetable.width - 20}px`"
              >
                <img src="../assets/station.svg" alt="station" class="station" />
                <div class="line" />
                <div class="rotated">
                  {{ formatDatehhmm(timetable.time) }}
                </div>
              </div>
            </div>
            <div class="road" :style="`width:${journey.totalWidth}px`">
              <img
                src="../assets/train.svg"
                alt="train"
                class="train"
                :style="`margin-left: ${-9 + (trainPosition < journey.totalWidth ? trainPosition : journey.totalWidth)}px;`"
              />
            </div>
          </div>
          <div v-else>
            <div v-for="timetable in journey.timetable" :key="timetable.station">
              {{ formatDatehhmm(timetable.time) + ': ' + timetable.station }}
            </div>
          </div>
        </td>
        <td v-if="innerWidth >= 1024">
          <div style="width:120px">{{ journey.nextStation }}</div>
        </td>
        <td>
          <div style="width:100px">{{ journey.train.name }}</div>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data: () => ({
    journeys: [],
    interval: undefined,
    trainPosition: 0,
    currentTime: undefined,
    trainsInTransit: [],
    innerWidth: 0
  }),
  created() {
    this.innerWidth = window.innerWidth
    window.addEventListener('resize', e => (this.innerWidth = e.currentTarget.innerWidth))

    axios.get('http://localhost:3000/journeys').then(({ data }) => {
      data.forEach(el => {
        el.timetable.forEach((timetable, i) => {
          timetable.width = this.calculateWidth(el.timetable[i - 1]?.time, timetable.time)
        })
        el.totalWidth = el.timetable.reduce((acc, el) => (acc += el.width), 0)
      })
      this.journeys = data
      this.currentTime = new Date('2020-06-04T09:00:00Z').getTime()
      this.trainsInTransit = this.journeys.map(t => t.train.name)
    })
  },
  methods: {
    formatDatehhmm(date) {
      return new Date(date).toJSON().substr(11, 5)
    },
    calculateWidth(startDate, endDate) {
      let start = new Date(startDate || endDate).getTime()
      let end = new Date(endDate).getTime()
      let totalMinutes = (end - start) / (1000 * 60)
      return totalMinutes * 5 //5px per min
    },
    start() {
      this.interval = setInterval(() => {
        this.trainPosition += 5 //every 500ms corresponds to 1 minute
        this.currentTime += 60 * 1000
        this.journeys.forEach(journey => {
          if (journey.totalWidth <= this.trainPosition) {
            this.trainsInTransit = this.trainsInTransit.filter(t => t != journey.train.name)
          } else {
            const nextStation = journey.timetable.find(s => new Date(s.time).getTime() > this.currentTime)
            journey.nextStation = nextStation.station
          }
        })
        if (this.trainPosition > 2 * 60 * 5) this.stop()
      }, 500)
    },
    stop() {
      clearInterval(this.interval)
      this.interval = undefined
    }
  }
}
</script>

<style scoped>
.station {
  width: 20px;
  height: 16px;
}
.line {
  position: relative;
  left: 9px;
  border-left: 2px solid black;
  height: 20px;
}
.timetable {
  margin-left: 30px;
  width: 610px;
  font-weight: normal;
}
table {
  margin-top: 10px;
  box-shadow: 0px 0px 16px #cccccc;
  border-spacing: 0px;
  width: 100%;
}
th {
  background-color: #ee3584;
  text-align: left;
  padding: 8px;
  color: whitesmoke;
}
td {
  padding: 4px;
}
tr:nth-child(odd) {
  background-color: #dddddd;
}
.road {
  height: 20px;
  margin: -56px 0px 40px 20px;
  background-image: url('../assets/road-texture.jpg');
  background-size: contain;
}
.rotated {
  writing-mode: vertical-lr;
  text-orientation: revert;
  transform: rotate(180deg);
}
.train {
  background-color: white;
  border-radius: 1rem;
  width: 16px;
  height: 16px;
  margin-left: -9px;
  margin-top: 1px;
  padding: 1px;
}
</style>
