<template>
    <div class="container">
        <template v-if="!isLoading">

            <template v-if="isCollectingData">
                <div class="loadingDataScreen">
                    <div class="loader"></div>
                </div>
            </template>

            <div class="SideBar">
                 <transition name="list">
                    <section v-show="modalOpen" class="modalSearch">
                        <section class="topModal">
                            <span class="material-icons md-48" @click="modalOpen = false">
                                close
                            </span>
                            <input type="text" v-model="city" @change="getCity(city)" placeholder="Search for places..." >
                        </section>

                        <section class="bottomModal">
                            <template v-if="gotCities.length">
                                <div v-for="i in gotCities" :key="i.name" @click="getWeather(i.coord.lon, i.coord.lat)">
                                    <p>{{i.name}}</p>
                                </div>
                            </template>
                            <template v-else>
                                <p>{{errorMessage}}</p>
                            </template>
                        </section>
                    </section>
                 </transition>

                <button @click="modalOpen = true">Search for places</button>

                <!-- <img class="SideBarBackground" src="../../assets/sidebar.svg" alt="background-sidebar"> -->
                <img :src="'http://openweathermap.org/img/wn/' + weather[0].weather_icon + '@4x.png'" alt="weather">

                <h1>{{weather[0].temp}}</h1>

                <p class="weatherState">{{weather[0].weather_name}}</p>

                <section class="footerSideBar">
                    <p>Today | {{date}} <br /><br /></p>
                        <span class="material-icons">location_on</span>
                        {{weather[0].name}}
                </section>

                <button :class="{ 'switchEuTablet': europe, 'switchUsTablet': !europe}" @click="[europe = !europe, europeConverter()]">
                    <template v-if="europe">
                        US
                    </template>
                    <template v-else>
                        EU
                    </template>
                </button>
            </div>

            <div class="weatherInfo">
                <button :class="{ 'switchEu': europe, 'switchUs': !europe}" @click="[europe = !europe, europeConverter()]">
                    <template v-if="europe">
                        US
                    </template>
                    <template v-else>
                        EU
                    </template>
                </button>
                <section class="weatherForecast">
                    <div v-for="i in weather.slice(1)" :key="i.date">
                        <p>{{ i.date | moment }}</p>
                        <img class="weatherIconSmall" :src="'http://openweathermap.org/img/wn/' + i.weather_icon + '@2x.png'" alt="weather">
                        <p>{{i.max_temp}} &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {{ i.min_temp }}</p>
                    </div>
                </section>

                <section class="weatherDetails">
                    <h3>Today's hightlights</h3>
                    <section class="detailsTop">
                        <div class="square">
                            <p>Wind status</p>
                            <p>{{ weather[0].wind_speed }}</p>
                        </div>
                        <div class="square">
                            <p>Humidity</p>
                            <p>{{ weather[0].humidity }}%</p>
                        </div>
                    </section>

                    <section class="detailsBottom">
                        <div class="rectangle">
                            <p>Visibility</p>
                            <p>{{ weather[0].visibility }}</p>
                        </div>
                        <div class="rectangle">
                            <p>Air pressure</p>
                            <p>{{ weather[0].air_pressure }}</p>
                        </div>
                    </section>
                </section>

                <section class="footer">
                    <p>created by <span style="color:#e94560;"><a href="https://pbielanin.pl" target="_blank">pbielanin</a></span></p>
                </section>
            </div>
        </template>

        <template v-else>
            <div class="loadingScreen">
                <div class="loader"></div>
            </div>
        </template>
    </div>
</template>

<script>

    import axios from "axios";
    import moment from "moment";

    export default {
        name: "Weather",
        data(){
            return{
                weather: [],
                currentWeather: [],
                date: "2000-01-01",
                isLoading: true,
                modalOpen: false,
                city: "",
                gotCities: [],
                errorMessage: "",
                isCollectingData: false,
                europe: true
            }
        },
        methods:{
            getCity(city){
                this.gotCities = [];
                if(city == ""){
                    this.errorMessage = "No informations about this city";
                } else{
                    axios.get("https://api.openweathermap.org/data/2.5/weather?q="+city+"&appid=c60a0f362625b9cdeae8771730c3ac1a")
                        .then( ({data}) => {
                            this.gotCities.push(data);
                        })
                        .catch( () => {
                            this.errorMessage = "No informations about this city";
                        });
                }
            },

            getWeather(lon, lat){
                this.isCollectingData = true;
                this.modalOpen = false;
                axios.get("https://api.openweathermap.org/data/2.5/onecall?lat="+lat+"&lon="+lon+"&exclude=minutely,hourly,alerts&units=metric&appid=c60a0f362625b9cdeae8771730c3ac1a")
                    .then( ({data}) => {
                        this.currentWeather = data;
                        this.europeConverter();
                        this.isCollectingData = false;
                    });
            },

            europeConverter(){
                this.weather = [];
                for(let i=0; i<=5 ; i++){
                    if (this.europe === true){
                        const wind_speed = Math.round(this.currentWeather.daily[i].wind_speed);
                        const visibility = Math.round(this.currentWeather.current.visibility/1000);
                        const air_pressure = this.currentWeather.daily[i].pressure;
                        const temp = Math.round(this.currentWeather.current.temp);
                        const min_temp = Math.round(this.currentWeather.daily[i].temp.min);
                        const max_temp= Math.round(this.currentWeather.daily[i].temp.max);
                        this.weather.push({
                            "temp": temp + " °C",
                            "visibility": visibility + " km",
                            "wind_speed": wind_speed.toString() + " km/h",
                            "air_pressure": air_pressure + " hPa",
                            "min_temp": min_temp + " °C",
                            "max_temp": max_temp + " °C",
                            "humidity": this.currentWeather.daily[i].humidity,
                            "weather_name": this.currentWeather.daily[i].weather[0].main,
                            "weather_icon": this.currentWeather.daily[i].weather[0].icon,
                            "date": this.currentWeather.daily[i].dt,
                            "name": this.gotCities[0].name
                        })
                    } else{
                        const wind_speed = Math.round(this.currentWeather.daily[i].wind_speed / 1.609344);
                        const visibility = Math.round(this.currentWeather.current.visibility * 0.00062137);
                        const air_pressure = this.currentWeather.daily[i].pressure;
                        const temp = Math.round(32 + (9/5) * this.currentWeather.current.temp);
                        const min_temp = Math.round(32 + (9/5) * this.currentWeather.daily[i].temp.min);
                        const max_temp= Math.round(32 + (9/5) * this.currentWeather.daily[i].temp.max);
                        this.weather.push({
                            "temp": temp + " °F",
                            "visibility": visibility + " miles",
                            "wind_speed": wind_speed.toString() + " mp/h",
                            "air_pressure": air_pressure + " mb",
                            "min_temp": min_temp + " °F",
                            "max_temp": max_temp + " °F",
                            "humidity": this.currentWeather.daily[i].humidity,
                            "weather_name": this.currentWeather.daily[i].weather[0].main,
                            "weather_icon": this.currentWeather.daily[i].weather[0].icon,
                            "date": this.currentWeather.daily[i].dt,
                            "name": this.gotCities[0].name
                        })
                    }
                };
            }
        },
        filters: {
            moment: function (date) {
                return moment.unix(date).format("ddd, Do MMM");
            }
        },
        created(){
            currentWeather:
                axios.get("https://api.openweathermap.org/data/2.5/onecall?lat=50.2584&lon=19.0275&exclude=minutely,hourly,alerts&units=metric&appid=c60a0f362625b9cdeae8771730c3ac1a")
                    .then( ({data}) => {
                        this.currentWeather = data;
                        this.isLoading = false;
                        this.gotCities.push({
                            "name": "Katowice"
                        });

                        this.europeConverter();
                    });

            date:
                this.date = moment().format("ddd, Do MMM");

        }
    }
</script>

<style lang="scss" scoped>
    @import '../../styles/Weather.scss';
</style>
