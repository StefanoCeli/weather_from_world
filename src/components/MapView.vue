<template>
  <div class="map-container">
    <div id="map" @click="clickWeather"></div>

      <div class="most-popular">
        <div class="dayli-info-pop" v-for="pop in popularLocationWeather" :key= pop.id>
          <h1>{{pop.name}}</h1>
          <div class="border">
            <p>Temperatura attuale: {{pop.main.temp}} °C</p>
            <p>Temperatura massima: {{pop.main.temp_max}} °C</p>
            <p>Temperatura minima: {{pop.main.temp_min}} °C</p>
            <p>Tempo atmosferico: {{pop.weather[0].description}}</p>
          </div>
        </div>
      </div>

    <div  class="weather-info">
      <div class="dayli-info"
      @click.prevent="active = index"
      v-for="(day,index) in days"
      :key="index"
      :class=" active === index ? '' : 'centerDate'"
      >
        <p v-if="active !== index">Clicca qui per le previsioni di:</p>
        <h2>{{formatDate(next5Days[index])}}</h2>
        <h2>{{city}}</h2>
        <div
        v-for="singleDay in day"
        :key="singleDay.dt"
        >
          <div v-if="active === index" class="border">
            <p>Ore: {{singleDay.dt_txt.substring(10,16)}}</p>
            <p>Temperatura: {{singleDay.main.temp}} °C</p>
            <p>Percepiti: {{singleDay.main.feels_like}} °C</p>
            <p>Tempo atmosferico: {{singleDay.weather[0].description}}</p>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import L from "leaflet";

export default {
  name: "MapView",

  data() {
    return {
        map: {},
        selectedLocation : {},
        selectedLocationWeather : {},
        popularLocationWeather: [],
        days:[],
        city:"",
        next5Days:[],
        marker: "",
        active: 0,
        lat: 41.894614646762655,
        lon: 12.481583476104666,
    }
  },

  mounted(){
    this.startMap();
  },

  methods: {

    markerMap(){
      const customIcon = L.icon({
      iconUrl: require("@/assets/marker.svg"), // Percorso dell'icona personalizzata
      iconSize: [32, 32], // Dimensioni dell'icona
      });
      //se il marker è già presente lo sostituisce con il successivo
      if(this.marker!="")  this.map.value.removeLayer(this.marker)
      this.marker = L.marker([this.lat, this.lon], { icon: customIcon }).addTo(this.map.value);
    },
    startMap() {
    this.map.value = L.map("map").setView([this.lat, this.lon], 6);
    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "© OpenStreetMap contributors",
    }).addTo(this.map.value);
    this.markerMap();
    this.fetchWeatherData(this.lat, this.lon);
    this.fetchWeatherDataPop();
    },

    async fetchWeatherData(lat,lon) {
      const apiKey = "API_KEY";
      const lang = "it";
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&units=metric&lang=${lang}&appid=${apiKey}`
      );
      const data = await response.json();
      this.selectedLocationWeather.value = data;
      console.log( this.selectedLocationWeather.value.list[0].main.temp )
      this.days= []
      this.city=this.selectedLocationWeather.value.city.name
      this.filterDays();
    },
    async fetchWeatherDataPop() {
      const apiKey = "b6673bd69190fed9ac36321c42d23d50";
      const lang = "it";
      const popLocation = [{lat:48.854386141406984, lon:2.347650803777937},{lat:51.505753732675124, lon:-0.12393951416015626},{lat:40.71188986623852, lon:-74.00974984065783},{lat:55.75011320248252, lon:37.61710102038162}]
      for (let i = 0; i < popLocation.length; i++) {
        const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?lat=${popLocation[i].lat}&lon=${popLocation[i].lon}&units=metric&lang=${lang}&appid=${apiKey}`
      );
      const data = await response.json();
      this.popularLocationWeather.push(data);
      console.log( this.popularLocationWeather)
      }
    },

    clickWeather() {
        this.map.value.on("click", (event) => {
            //settaggio latitudine e longitudine
            this.lat = event.latlng.lat;
            let a = event.latlng.lat;
            this.lon = event.latlng.lng;
            let b = event.latlng.lng;
            this.selectedLocation.value = { a, b };
            //aggiunta marker quando si clicca sulla mappa
            this.markerMap();
            //richiamo la funzione per aggiornare i dati a schermo
            this.fetchWeatherData(this.lat, this.lon);
      });
    },
    filterDays() {
       function formatDateString(date) {
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        return `${year}-${month}-${day}`;
      }

      function getDatesForNext5Days() {
        const today = new Date();
        const dates = [];

        for (let i = 0; i < 5; i++) {
          const nextDay = new Date(today);
          nextDay.setDate(today.getDate() + i);

          // Gestione del passaggio al mese successivo
          if (nextDay.getMonth() !== today.getMonth()) {
            nextDay.setMonth(today.getMonth());
            nextDay.setDate(today.getDate() + i);
          }

          dates.push(formatDateString(nextDay));
        }

        return dates;
      }

      this.next5Days = getDatesForNext5Days();
      
      for (let i = 0; i < 5; i++) {
        this.days.push(this.selectedLocationWeather.value.list.filter(item => item.dt_txt.includes(this.next5Days[i])))
      }

    },
    formatDate(date){
      const updateDate = new Date(date)
      const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
      console.log("ciaone " + updateDate)
      var str = updateDate.toLocaleDateString(undefined, options);
      var splitStr = str.toLowerCase().split(" ");
      for (var i = 0; i < splitStr.length; i++) {
        splitStr[i] = splitStr[i].charAt(0).toUpperCase() + splitStr[i].substring(1);
      }
      return splitStr.join(" ");
    }
    
  },
};
</script>

<style lang="scss" scoped>
.map-container {
  position: relative;
  width: 100%;
  height: 100vh;
  color: white;
  font-size: 20px;
  background-color: rgb(160, 161, 160);
  background-image: url("../assets/w-image.jpg");
  #map {
    height: calc(80% - 100px);
    width: 80%;
    z-index: 1;
    position: absolute;
    right: 0;
    bottom: 20%;
    
  }
  .most-popular {
      width: 20%;
      height: calc(100vh - 20%);
      display: flex;
      flex-direction: column;
      justify-content:space-between;
      overflow: auto;
      .dayli-info-pop {
      border: 1px solid black;
      background-color: rgba(0, 0, 0, 0.3);
      height: 100%;
      width: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      .border {
        background-color: rgba($color: #000000, $alpha: 0.3);
        width: 95%;
        margin: 5px auto;
      }
      p {
        padding: 10px 0;
      }
    }
    
  }

  .weather-info {
    background-color: rgba(0, 0, 0, 0.3);
    width: 100%;
    height: 20%;
    border-radius: 5px;
    position: absolute;
    bottom: 0;
    left: 0;
    z-index: 2;
    display: flex;
    .dayli-info {
      border: 1px solid black;
      height: 100%;
      overflow: auto;
      width: 25%;
      cursor: pointer;
      .border {
        background-color: rgba($color: #000000, $alpha: 0.3);
        width: 95%;
        margin: 10px auto;
      }
      p{
        padding: 10px 0;
      }
    }
  }
  .centerDate{
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
}
</style>
