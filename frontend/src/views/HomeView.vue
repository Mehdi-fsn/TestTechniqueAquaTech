<template>
  <div class="container">
    <nav>
      <p><strong>Test Technique Aquatech</strong></p>
      <p>Génération des données :</p>
      <button type="button" class="btn" @click="generateData">Genérer les données</button>
      <p>Refresh des données :</p>
      <button type="button" class="btn" @click="refreshData">Refresh</button>
      <form @submit.prevent="updateData">
        <label for="datemin" class="form-label">Date minimum :</label>
        <input type="datetime-local" class="datemin" v-model="datemin" />
        <label for="datemax" class="form-label">Date maximum :</label>
        <input type="datetime-local" class="datemax" v-model="datemax" />
        <button type="submit" class="btn submit">
          Mettre à jour les données
        </button>
      </form>
    </nav>
    <div id="canvas">
      <canvas id="eau"></canvas>
      <canvas id="pression"></canvas>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Chart from "chart.js/auto";

export default {
  name: "HomeView",
  data() {
    return {
      dataEau: null,
      dataPression: null,
      datemin: "2022-11-28 00:00:00",
      datemax: "2022-12-04 23:59:00",
    };
  },
  async mounted() {
    // Récupération des données
    let dataE = await axios
      .post(
        "http://127.0.0.1:5000/getDataEau?",
        `date_min=${this.datemin}&date_max=${this.datemax}`
      )
      .then((result) => {
        return result.data;
      })
      .catch((err) => console.log(err));

    let dataP = await axios
      .post(
        "http://127.0.0.1:5000/getDataPression?",
        `date_min=${this.datemin}&date_max=${this.datemax}`
      )
      .then((result) => {
        return result.data;
      })
      .catch((err) => console.log(err));

    let dataEau = dataE.map((d) => d.cm);
    let dataPression = dataP.map((d) => d.bar);
    let dataDate = dataP.map((d) => d.date);
    console.log(dataEau);
    console.log(dataPression);
    console.log(dataDate);

    // Création des graphiques

    this.dataEau = this.constructChart(
      document.getElementById("eau"),
      "line",
      dataDate,
      "Niveau d’eau",
      dataEau,
      "#107AD7"
    );
    this.dataPression = this.constructChart(
      document.getElementById("pression"),
      "line",
      dataDate,
      "Pression",
      dataPression,
      "#E84B15"
    );
  },
  methods: {
    constructChart(element, type = "line", labels, label, data, color) {
      return new Chart(element, {
        type: type,
        data: {
          labels: labels,
          datasets: [
            {
              label: label,
              data: data,
              borderWidth: 1,
              borderColor: color,
              backgroundColor: "#B8B9B4",
              lineTension: 0.3,
            },
          ],
        },
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: false,
            },
          },
          xAxes: [
            {
              type: "time",
            },
          ],
          interactions: {
            mode: "x",
          },
        },
      });
    },
    async updateData() {
      // Conditions de refresh (datemin < datemax)
      if (this.datemin > this.datemax) {
        this.$toast.error(
          "La date minimum doit être inférieure à la date maximum"
        );
        return;
      }
      if (
        this.datemin < "2022-11-28 00:00:00" ||
        this.datemax > "2022-12-04 23:59:00"
      ) {
        this.$toast.error(
          "Veuillez choisir une date comprise entre le 2022-11-28 00:00:00 et le 2022-12-04 23:59:00"
        );
        return;
      }

      // Récupération des données
      let dataE = await axios
        .post(
          "http://127.0.0.1:5000/getDataEau?",
          `date_min=${this.datemin}&date_max=${this.datemax}`
        )
        .then((result) => {
          return result.data;
        })
        .catch((err) => console.log(err));

      let dataP = await axios
        .post(
          "http://127.0.0.1:5000/getDataPression?",
          `date_min=${this.datemin}&date_max=${this.datemax}`
        )
        .then((result) => {
          return result.data;
        })
        .catch((err) => console.log(err));

      let dataEau = dataE.map((d) => d.cm);
      let dataPression = dataP.map((d) => d.bar);
      let dataDate = dataP.map((d) => d.date);
      console.log(dataEau);
      console.log(dataPression);
      console.log(dataDate);

      // Suprresion des anciens graphiques
      this.dataEau.destroy();
      this.dataPression.destroy();

      // Création des nouveaux
      this.dataEau = this.constructChart(
        document.getElementById("eau"),
        "line",
        dataDate,
        "Niveau d’eau",
        dataEau,
        "#107AD7"
      );
      this.dataPression = this.constructChart(
        document.getElementById("pression"),
        "line",
        dataDate,
        "Pression",
        dataPression,
        "#E84B15"
      );

      this.$toast.success("Update, OK !");
    },
    async refreshData() {
      await axios
        .post("http://127.0.0.1:5000/refresh")
        .catch((err) => console.log(err));

      this.updateData()
    },
    async generateData() {
        await axios
        .post("http://127.0.0.1:5000/generator")
        .then((response) => {
          if(response.data.result == "error") {
            this.$toast.error("Vous avez déja génrérer des données !")
          }
          else {
            this.updateData()
          }
        })
        .catch((err) => console.log(err));        
    }
  },
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: space-between;
}

nav {
  background-color: rgba(20, 20, 20, 0.8);
  box-shadow: 4px 7px 10px rgba(0, 0, 0, 0.4);
  color: white;
  height: auto;
  border-radius: 10px;
}

form {
  display: flex;
  flex-direction: column;
  background-color: rgba(255, 255, 255, 0.7);
  color: black;
  margin: 10px 10px;
  border-radius: 10px;
}

form * {
  margin: 7px 7px;
}

#canvas {
  flex: 1;
}

canvas {
  background-color: rgba(255, 255, 255, 0.7);
  box-shadow: 4px 7px 10px rgba(0, 0, 0, 0.4);
  border-radius: 10px;
  margin: 0 10px 10px 10px;
}
</style>