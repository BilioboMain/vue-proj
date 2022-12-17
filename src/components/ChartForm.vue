<template>
  <form @submit.prevent action="" class="date-form">
    <div class="form__item">
    </div>
    <div class="form__item">
      <label class="form__label" for="sensor-selector">Датчик</label>
      <select v-model="selected">
        <option v-for="sensor in sensors" :key="sensor.name_sensor">
          {{ sensor.name_sensor }}
        </option>
      </select>
      <div>Selected: {{ selected }}</div>
    </div>
    <div @change="correctDates" class="date-fields">
      <div class="form__item">
        <label class="form__label" for="date-start">Начало периода</label>
        <input
          v-model="dateStart"
          :max="formatDate(today)"
          type="date"
          name="date-start"
          id="date-begin"
          class="form__field"
        />
      </div>
      <div class="form__item">
        <label class="form__label" for="date-end">Конец периода</label>
        <input
          v-model="dateEnd"
          :max="formatDate(today)"
          type="date"
          name="date-end"
          id="date-end"
          class="form__field"
        />
      </div>
    </div>
    <!-- /.date-fields -->

    <!-- /.form--field -->
    <input
      @click="sendData"
      :disabled="isEmpty"
      type="submit"
      value="Получить данные"
      class="form__submit button"
    />
    <!-- /.data-form__submit button -->
  </form>
  <!-- /.date-form -->
</template>

<script>
import axios from "axios";
// const data = require('src/assets/datasets/data1.json');
export default {
  props: {},
  data() {
    return {
      sensorType: "",
      sensorId: "",
      dateStart: "",
      dateEnd: "",
      DAY_IN_MS: 86400000,
      sensors:[],
      selected: 'A',
    };
  },
  methods: {
    sendData() {
      this.$emit("sendData", {
        sensorsData: this.chosenSensorId(this.sensors),
        selected: this.chosenSensorId(this.sensors),
        // sensorName:
        sensorType: this.selected,
        dateStart: this.dateStart.split('-').reverse().join('-'),
        dateEnd: this.dateEnd.split('-').reverse().join('-'),
        period: `${this.dateStart}:${this.dateEnd}`,
      });
      console.log(
        "Тип",
        this.chosenSensorId(this.sensors),
        "id:",
        "\nНачало периода",
        this.dateStart.split('-').reverse().join('-'),
        "Конец периода",
        this.dateEnd.split('-').reverse().join('-')
      );
    },
    getData() {
      console.log("data");
    },
    sensorsData(sensors){
       return sensors.map(({name_sensor,id_sensor}) => ({ [name_sensor]: id_sensor}));
    },
    chosenSensorId(sensors){
      console.log('did')
      for(let key in sensors){
        if (sensors[key].name_sensor == this.selected){
          return sensors[key].id_sensor
        }
      }
    },
    formatDate(date) {
      return date.toISOString().match(/[\d-]+/)[0];
    },
    compareDates(date1, date2) {
      const difference = Date.parse(date1) - Date.parse(date2);
      if (difference < 0) {
        return -1;
      } else if (difference > 0) {
        return 1;
      } else return 0;
    },
    correctDates() {
      if (this.compareDates(this.dateStart, this.dateEnd) > 0) {
        console.log(
          "dates swaped",
          this.compareDates(this.dateStart, this.dateEnd)
        );
        this.dateStart = this.formatDate(
          new Date(Date.parse(this.dateEnd) - this.DAY_IN_MS)
        );
      }
    },
  },
  computed: {
    today() {
      return new Date();
    },
    isEmpty() {
      return !(
        this.sensors &&
        this.dateStart &&
        this.dateEnd
      );
    },
  },
  mounted() {
    axios
      .get('http://194.58.98.82:8080/sensor/allSensors')
      .then(response => (this.sensors = response.data,
      console.log(response.data)
      ))
  },
  beforeMount() {
    const currDate = this.today;
    const weekBefore = new Date(currDate - this.DAY_IN_MS * 7);

    this.dateStart = this.formatDate(weekBefore);
    this.dateEnd = this.formatDate(currDate);
  },
};
</script>
