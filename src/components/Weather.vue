
<template>
  <div class="container">
    <div class="header">
      <div class="header__inputs">
        <span class="header__input-text">Latitude</span>
        <span class="header__input-text">Longitude</span>
        <span class="header__input-text">Temperature Unit</span>
        <span class="header__input-text">Windspeed Unit</span>
        <span class="header__input-text">Precipation Unit</span>
        <input v-model="lat" class="header__input">
        <input v-model="long" class="header__input">
        <select v-model='tempUnit' class="header__input">
          <option value="fahrenheit">Fahrenheit</option>
          <option value="celcius">Celcius</option>
        </select>
        <select v-model='windspeedUnit' class="header__input">
          <option value="mph">Mph</option>
          <option value="kmh">Kmh</option>
          <option value="kn">Knots</option>
          <option value="ms">M/s</option>
        </select>
        <select v-model="precipitationUnit" class="header__input">
          <option value="inch">In.</option>
          <option value="mm">mm</option> 
        </select>
      </div>
      <button class="btn__weather" @click="getHourlyWeather(), getDailyWeather()">Go</button>
    </div>
    <div class="daily">
      <template v-for="(item, index) in dailyTime" :key="item">
        <template v-if="index == 0"></template>
        <div v-else class="daily__item">
          <img class="daily__img" :src="getWeatherType(dailyCode[index])">
          <span class="daily__time">{{dailyTime[index].slice(5)}}</span>
          <span class="daily__temp">High: {{dailyTempMax[index]}}{{tempUnitUsed}}<br/>Low: {{dailyTempMin[index]}}{{tempUnitUsed}}</span>
          <span class="daily__sunrise">Sunrise: {{dailySunrise[index].slice(11)}}<br/>Sunset: {{dailySunset[index].slice(11)}}</span>
          <span v-if="precipitationUnitUsed == 'inch'" class="daily__precipitation">Total Precipitation: {{dailyPrecipitation[index]}} {{precipitationUnitUsed}}es</span>
          <span v-else class="daily__precipitation">Total Precipitation: {{dailyPrecipitation[index]}} {{precipitationUnitUsed}}</span>
        </div>
      </template>
    </div>
    <div class="today" v-if="dailyTime[0] && hourlyTime[0]">
      <div class="today__info">
        <div class="today__info-container-img">
          <img class="today__info-img" :src="getWeatherType(currWeathercode)">
          <span class="today__info-temp">High: {{dailyTempMax[0]}}{{tempUnitUsed}}</span>
          <span class="today__info-temp">Low: {{dailyTempMin[0]}}{{tempUnitUsed}}</span>
          <span class="today__info-humidity">Humidity: {{hourlyHumidity[hourOffset]}}%</span>
          
        </div>  
        <div class="today__info-time">{{getDay(dailyTime[0])}}<br/>{{dailyTime[0].slice(5)}}</div>
        <div class="today__info-sunrise">Sunrise: {{dailySunrise[0].slice(11)}}<br/>Sunset: {{dailySunset[0].slice(11)}}</div>
        <div v-if="precipitationUnitUsed == 'inch'" class="today__info-precipitation">Total Precipitation: {{dailyPrecipitation[0]}} {{precipitationUnitUsed}}es</div>
        <div v-else class="today__info-precipitation">Total Precipitation: {{dailyPrecipitation[0]}} {{precipitationUnitUsed}}</div>
        <div class="today__info-windspeed"><div class="today__info-windspeed-arrow" :style="{transform: 'rotate('+currWindDir+'deg)'}" >↑</div> 
          <br/><div>Windspeed: {{currWindSpeed}} {{windspeedUnitUsed}} {{getWindDirection(currWindDir)}}</div>
          <br/><div class="today__info-windgusts">Max Wind Gusts: {{dailyGustsMax[0]}} {{windspeedUnitUsed}}</div>
        </div>
        <div class="today__info-cloudcover">Cloudcover: {{hourlyCloudcover[hourOffset]}}%</div>
      </div>
      <div class="today__hourly">
        <button v-if="hourlyNum == 0" class="btn__hourly-left" disabled>❮</button>
        <button v-else class="btn__hourly-left" @click="hourlyNum--">❮</button>
        <template v-for="index in 6" :key="index">
          <div class="today__hourly-container">
            <img class="today__hourly-img" :src="getWeatherType(hourlyCode[index+hourlyNum+hourOffset])">
            <span class="hourly__time">{{hourlyTime[index+hourlyNum+hourOffset].slice(11)}} {{hourlyTemp[index+hourlyNum+hourOffset]}}{{tempUnitUsed}}</span>
            <span v-if="precipitationUnitUsed == 'inch'" class="hourly__precipitation">{{hourlyPrecipitation[index+hourlyNum+hourOffset]}} {{precipitationUnitUsed}}es</span>
            <span v-else class="hourly__precipitation">{{hourlyPrecipitation[index+hourlyNum+hourOffset]}} {{precipitationUnitUsed}}</span>
          </div> 
        </template>
        <button v-if="hourlyNum >= 48" class="btn__hourly-right" disabled>❯</button>
        <button v-else class="btn__hourly-right" @click="hourlyNum++">❯</button>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Weather',
  data(){
    return{
      days: ['Sunday', 'Monday', 'Teusday', 'Wednsday', 'Thursday', 'Friday', 'Saturday'],
      address: 'Forest Dr',
      city: 'Columbia',
      state: 'SC',
      tempUnit: 'fahrenheit',
      tempUnitUsed: '',
      windspeedUnit: 'mph',
      windspeedUnitUsed: '',
      precipitationUnit: 'inch',
      precipitationUnitUsed: '',
      lat: 34.14,
      long: -80.89,
      hourlyTemp: [],
      hourlyPrecipitation: [],
      hourlyTime: [],
      hourlyWind: [],
      hourlyCode: [],
      hourlyHumidity: [],
      hourlyCloudcover: [],
      hourlyNum: 0,
      dailyTempMax: [],
      dailyTempMin: [],
      dailyTime: [],
      dailySunrise: [],
      dailySunset: [],
      dailyPrecipitation: [],
      dailyWindMax: [],
      dailyGustsMax: [],
      dailyCode: [],
      currTime: '',
      currWeathercode: 0,
      currTemp: 0,
      currWindDir: 0,
      currWindSpeed: 0,

    }
  },
  mounted(){
    //this.getCoords();
    this.getHourlyWeather();
    this.getDailyWeather();
    this.getCurrentWeather();
  },
  methods: {
    getCoords(){
      let url = "https://geocoding.geo.census.gov/geocoder/locations/address?street=" + this.address + "&city=" + this.city + "&state=" + this.state + "&benchmark=2020&format=json"
      axios.get(url).then( response => {
        console.log("coords", response)
      })
    },
    getHourlyWeather(){
      let url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&hourly=temperature_2m,precipitation,winddirection_10m,windspeed_10m,weathercode,relativehumidity_2m,cloudcover&temperature_unit=" +this.tempUnit + "&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York";
      if(this.tempUnit == "celcius"){
        url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&hourly=temperature_2m,precipitation,winddirection_10m,windspeed_10m,weathercode,relativehumidity_2m,cloudcover&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York"
      }
      axios.get(url).then( response => {
        this.hourlyTemp = response.data.hourly.temperature_2m;
        this.hourlyPrecipitation = response.data.hourly.precipitation;
        this.hourlyTime = response.data.hourly.time;
        this.hourlyWind = response.data.hourly.windpseed_10m;
        this.hourlyWindDirection = response.data.hourly.winddirection_10m;
        this.hourlyCode = response.data.hourly.weathercode;
        this.hourlyCloudcover = response.data.hourly.cloudcover;
        this.hourlyHumidity = response.data.hourly.relativehumidity_2m;

        this.tempUnitUsed = response.data.hourly_units.temperature_2m;
        this.precipitationUnitUsed = response.data.hourly_units.precipitation;
        this.windspeedUnitUsed = response.data.hourly_units.windspeed_10m;
        console.log("hourly", response);
      })
    },
    getDailyWeather(){
      let url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&daily=temperature_2m_max,temperature_2m_min,sunrise,sunset,precipitation_sum,windspeed_10m_max,windgusts_10m_max,weathercode&temperature_unit=" +this.tempUnit + "&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York";
      if(this.tempUnit == "celcius"){
        url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&daily=temperature_2m_max,temperature_2m_min,sunrise,sunset,precipitation_sum,windspeed_10m_max,windgusts_10m_max,weathercode&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York"
      }
      axios.get(url).then( response => {
        this.dailyTempMax = response.data.daily.temperature_2m_max;
        this.dailyTempMin = response.data.daily.temperature_2m_min;
        this.dailyTime = response.data.daily.time;
        this.dailyPrecipitation = response.data.daily.precipitation_sum;
        this.dailySunrise = response.data.daily.sunrise;
        this.dailySunset = response.data.daily.sunset;
        this.dailyGustsMax = response.data.daily.windgusts_10m_max;
        this.dailyWindMax = response.data.daily.windspeed_10m_max;
        this.dailyCode = response.data.daily.weathercode;
        console.log("daily", response);
      })
    },
    getCurrentWeather(){
      let url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&current_weather=true&temperature_unit=" +this.tempUnit + "&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York";
      if(this.tempUnit == 'celcius'){
        url = "https://api.open-meteo.com/v1/forecast?latitude=" + this.lat + "&longitude=" + this.long + "&current_weather=true&windspeed_unit=" + this.windspeedUnit + "&precipitation_unit=" + this.precipitationUnit + "&timezone=America%2FNew_York";
      }
      axios.get(url).then(response => {
        this.currTime = response.data.current_weather.time;
        this.currWeathercode = response.data.current_weather.weathercode;
        this.currTemp = response.data.current_weather.temperature;
        this.currWindDir = response.data.current_weather.winddirection;
        this.currWindSpeed = response.data.current_weather.windspeed;
        console.log("current", response)
        this.getDay(response.data.current_weather.time);
      })
    },
    getWeatherType(code){
      if(code == 0 || code == 1 || code == 2){
        return require("../assets/sun.png");
      }else if(code == 3 || code == 45 || code == 48){
        return require("../assets/clouds.png");
      }else if(code == 51 || code == 53 || code == 55 || code == 56 || code == 57 || code == 61 || code == 63 || code == 65 || code == 66 || code == 67 || code == 80 || code == 81 || code == 82){
        return require("../assets/rain.png");
      }else{
        return require("../assets/thunder.png");
      }
    },
    getWindDirection(val){
      if(val <= 15 || val >= 345){
        return 'N';
      }else if(val <= 75 && val > 15){
        return 'NE';
      }else if(val <= 105 && val > 75){
        return 'E';
      }else if(val <= 165 && val > 105){
        return 'SE';
      }else if(val <= 195 && val > 165){
        return 'S';
      }else if(val <= 255 && val > 195){
        return 'SW';
      }else if(val <= 285 && val > 255){
        return 'W';
      }else if(val <= 345 && val > 285){
        return 'NW';
      }
    },
    getDay(time){
      let newdate = new Date(time);
      return this.days[newdate.getDay()];
    },
  },
  computed: {
    hourOffset(){
      let offset = this.currTime.slice(11, 13);
      if(offset.charAt(0) == 0){
        return parseInt(offset.slice(1));
      }else{
        return parseInt(offset)
      }  
    },
    arrowRotate(){
      return {
        transform: 'rotateY('+this.currWindDir+'deg)'
      }
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
html{
  font-size: 10px;
}
.container{
  position: absolute;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100%;
  background-image: linear-gradient(to bottom right, aqua, blue);

  display: grid;
  grid-template-columns: 1fr 2.5fr 1.5fr;
  grid-template-rows: 7rem 1fr 10rem;
}
/*/////////////////
Header


/////////////////*/
.header{
  grid-column: 1/ -1;
  grid-row: 1;
  background-color: rgb(25, 25, 200);

  display: grid;
  grid-template-columns: 1fr 2.5fr 1.5fr;
}
.header__inputs{
  grid-column: 2;
  grid-row: 1;
  display: grid;
  grid-template-rows: repeat(2,1fr);
  grid-template-columns: repeat(5,1fr);
  align-items: center;
}
.header__input{
  margin-left: 2rem;
  margin-right: 2rem;
  height: 2rem;
  width: 7rem;
  padding-left: 1rem;

  grid-row: 2;
}
.header__input-text{
  color: white;

  align-self: end;
}
.btn__weather{
  border-radius: 20%;
  height: 3rem;
  width: 7rem;
  background-color: #fff;
  margin-top: 2rem;

  grid-row: 1;
  grid-column: 3;
  align-self: center;
  justify-self: start;
}

/*///////////////
Daily


///////////////*/
.daily{
  background-color: rgb(0, 255, 255,.3);
  margin-right: 8rem;
  color: black;

  grid-row: 2;
  grid-column: 3;

  display: grid;
  grid-template-rows: repeat(6,1fr);
}
.daily__item{
  background-color: aliceblue;
  margin: .5rem;
  box-shadow: 0 .5rem 1rem rgba(0,0,0,.2);
  transition: all .2s;

  display: grid;
  grid-template-columns: .6fr 1fr 1fr;
  grid-template-rows: 2fr 1fr;
  align-items: center;
}
.daily__item:hover{
  transform: scale(1.05);
  box-shadow: 0 1rem 2rem rgba(0,0,0,.2);
}
.daily__img {
  width: 4rem;
  height: 4rem;
  grid-column: 1;
  grid-row: 1;
  align-self: center;
  justify-self: center;
}
.daily__time{
  grid-row: 2;
  grid-column: 1;
}
.daily__precipitation{
  grid-column: span 2;
}

/*////////////////
Center


////////////////*/
.today{
  grid-row: 2;
  grid-column: 2;
  background-color: rgb(255,255,255,.7);

  display: grid;
  grid-template-rows: 1fr 10rem;
}
.today__info{
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  align-items: center;
}
.today__info-container-img{
  grid-column: 1;
  grid-row: 1 / span 3;
  align-self: center;
  justify-self: center;
  background-color: aliceblue;
  padding: 2rem;
  box-shadow: 0 1rem 2rem rgba(0,0,0,.2);
  transition: all .2s;

  display: flex;
  flex-direction: column;
}
.today__info-container-img:hover{
  transform: scale(1.05);
}
.today__info-img{
  height: 12rem;
  width: 12rem;
  margin-bottom: 2rem;
}
.today__info-time{
  font-size: 3rem;
  font-weight: 700;
}
.today__info-temp{
  font-size: 2rem;
  font-weight: 500;
}
.today__info-humidity{
  font-size: 1.2rem;
  font-weight: 500;
}
.today__info-sunrise{
  grid-column: 3;
  grid-row: 3;
  font-size: 1.5rem;
  font-weight: 500;
}
.today__info-precipitation{
  font-size: 1.5rem;
  font-weight: 500;
}
.today__info-windspeed{
  font-size: 1.5rem;
  font-weight: 500;
  line-height: 2rem;

  grid-row: 1;
  grid-column: 3;
}
.today__info-windspeed-arrow{
  font-size: 4rem;
  position: relative;
  top: 1.5rem;
}
.today__info-windgusts{
  font-size: 1rem;
}
.today__info-cloudcover{
  font-size: 1.5rem;
  font-weight: 500;
}
.today__hourly{
  border: 1px solid black;
  display: grid;
  grid-template-columns: 3rem repeat(6, 1fr) 3rem;
}
.today__hourly-container{
  background-color: aliceblue;
  box-shadow: 0 1rem 2rem rgba(0,0,0,.2);
  margin: .5rem;
  border-radius: 10%;
  transition: all .2s;

  display: flex;
  flex-direction: column;
}
.today__hourly-container:hover{
  transform: scale(1.05);
}
.today__hourly-img{
  align-self: center;
  justify-self: center;
  margin-top: 1rem;
  height: 5rem;
  width: 5rem;
}
.btn__hourly-left{
  grid-column: 1;
  font-size: 2rem;
}
.btn__hourly-right{
  grid-column: 8;
  font-size: 2rem;
}
.btn__hourly[disabled]{
  background-color: darkgray;
  
}
</style>
