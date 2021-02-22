<template>
  <div id="app" :class="typeof weather.main!='undefined' && weather.main.temp>16 ?
    'warm':''">
    <main> 
      <h1>You may search a location or scroll the map to get the weather</h1>
      <div class="search-box">
        <input type="text" class="search-bar" placeholder="search..." 
        v-model="query"
        @keypress="fetchWeather">
      </div>
      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{weather.name}},{{weather.sys.country}}</div>
          <div class="date">{{dateBuilder()}}</div>
        </div>
        <div class="weather-box">
          <div class="temp">{{Math.round(weather.main.temp)}}°c</div>
          <div class="weather">{{weather.weather[0].main}} </div>
        </div>
        <br><br>
      </div>
      <div class="mapcomp">
          <l-map
          v-if="showMap"
          :zoom="zoom"
          :center="center"
          :options="mapOptions"
          style="height: 80%"
          @update:center="centerUpdate"
          @update:zoom="zoomUpdate"
        >
          <l-tile-layer
            :url="url"
            :attribution="attribution"
          />
        </l-map>
      </div>
    </main>
  </div>
</template>

<script>
import { latLng } from "leaflet";
import { LMap, LTileLayer } from "vue2-leaflet";
export default {
  name: 'App',
  components: {
    LMap,
    LTileLayer,
  },
  data(){
    return{
      //weather data
      api_key: '4243d879dd010783de0d2852a8b9ab5d',
      url_base: 'https://api.openweathermap.org/data/2.5/',
      query:'',
      weather:{},
      //map data
      zoom: 13,
      center: latLng(32.81079451181896, 34.982786178588874),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
        '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 11.5,
      currentCenter: latLng(47.41322, -1.219482),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5
      },
      showMap: true
      
    }
  },
 methods:{
   fetchWeather(e){
     if(e.key=="Enter"){
       fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
       .then(res=>{
         if(res.ok){
          return res.json();
         }
         else{
           alert("Input is invalid");
           return res.json();
         }
       }).then(this.setResult)
     }
   },
    fetchWeatherMap(){
       fetch(`${this.url_base}weather?lat=${this.currentCenter.lat}&lon=${this.currentCenter.lng}&units=metric&APPID=${this.api_key}`)
       .then(res=>{
         if(res.ok){
          return res.json();
         }
         else{
           alert("Input is invalid");
           return res.json();
         }
       }).then(this.setResult)
   },
   setResult(results){
     this.weather=results;
     this.center=latLng(results.coord.lat, results.coord.lon);
    },
   dateBuilder(){
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day} ${date} ${month} ${year}`;
   },
       zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
      this.fetchWeatherMap();
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    innerClick() {
      alert("Click!");
    }
 }

}
</script>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
}
body{
  font-family: 'montserrat',sans-serif;
}
#app{
  background-image: url(./assets/cold-bg.jpg);
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
main{
  min-height: 100vh;
  padding:25px;
  background-image:linear-gradient(to bottom,rgba(0,0,0,0.25),rgba(0,0,0,0.75));
}
.search-box{
  width:100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}
.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.location-box .date {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}
.weather-box {
  text-align: center;
}
.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color:rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .weather {
  color: #FFF;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
#app.warm {
  background-image: url(./assets/warm-bg.jpg);
}
.mapcomp{
  display: flex;
  justify-content: center;
  align-content: center;
  height: 500px;
  width: 100%;
}
h1{
  text-align: center;
  padding: 1%;
  font-family: 'montserrat',sans-serif;
  color: #FFF;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
</style>
