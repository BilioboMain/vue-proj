<template>
  <header class="container">
    <nav-block/>
  </header>
  <main class="main-container">
    <section class="flex-container">
      <div class="graph-block">
        <chart-form @sendData="requestData"/>
        <line-chart v-if="loaded" :chart-data="dataset"/>
      </div>
      <div class="graph-block">
        <chart-form @sendData="requestData"/>
        <line-chart v-if="loaded" :chart-data="dataset"/>
      </div>
      <div class="graph-block">
        <chart-form @sendData="requestData"/>
        <line-chart v-if="loaded" :chart-data="dataset"/>
      </div>
      <div class="graph-block">
        <chart-form @sendData="requestData"/>
        <line-chart v-if="loaded" :chart-data="dataset"/>
      </div>
      <!-- /.chart-cont -->
      <!-- <bar-chart v-if="loaded" :chart-data="dataset" /> -->
    </section>
    <div class="picture"></div>
  </main>
  <hr/>

  <footer class="container">
    <p class="footer-text">Проект мониторинга параметров здания "Монитор"</p>
    <!-- /.footer-text -->
  </footer>
</template>

<script>
// @ is an alias to /src
import axios from "axios";
import NavBlock from "@/components/NavBlock.vue";
import ChartForm from "@/components/ChartForm.vue";
import BarChart from "@/components/BarChart.vue";
import LineChart from "@/components/LineChart.vue";

export default {
  name: "HomeView",
  components: {
    NavBlock,
    ChartForm,
    // eslint-disable-next-line vue/no-unused-components
    BarChart,
    LineChart,
  },
  data() {
    return {
      dataPakage: "vue",
      dataPakageName: "",
      period: "",
      chartType: "",
      selected:"",
      downloads: [],
      labels: [],
      loaded: false,
      showError: true,
      sensorId:"",
      sensorsData:[],
      errorMessage: "Please enter a package",
    };
  },
  computed: {
    dataset() {
      return {
        labels: this.labels,
        datasets: [
          {
            label: "",
            borderColor: "#1C6DD0",
            pointBackgroundColor: "white",
            borderWidth: 1,
            pointBorderColor: "#1C6DD0",
            backgroundColor: "transparent",
            data: this.downloads,
          },
        ],
      };
    },
  },
  mounted() {
    // this.labels = [1, 2, 3];
    // this.downloads = ["l1", "l2", "l3"];
  },
  methods: {
    getFormData(formData) {
      console.log("got form data", formData);
      this.chartType = formData.sensorType;
      this.period = formData.period;
      this.sensorsData = formData.sensorsData
      this.selected = formData.selected
      this.dateStart = formData.dateStart
      this.dateEnd = formData.dateEnd
      console.log( `http://194.58.98.82:8080/measure/${this.selected}/${this.dateStart}/${this.dateEnd}`)

    },
    sensorIdData(data){
        data.forEach((el)=>{
          if (`${this.selected}` in el){
            return el[this.selected]
          }
        }
        )
    },
    resetState() {
      this.loaded = false;
      this.showError = false;
    },
    requestData(formData) {
      if (
          this.dataPakage === null ||
          this.dataPakage === "" ||
          this.dataPakage === "undefined"
      ) {
        this.showError = true;
        return;
      }
      this.resetState();
      this.getFormData(formData);

      axios
          .get(
              `http://194.58.98.82:8080/measure/${this.selected}/${this.dateStart}/${this.dateEnd}`
          )
          .then((response) => {
            console.log(response.data);
            this.downloads = response.data.value.map(
                (download) => download.value
            );
            this.labels = response.data.value.map((download) => download.day);
            this.dataPakageName = response.data.dataPakage;
            this.loaded = true;
            console.log(this.downloads)
          })
    },
  },
};
</script>
