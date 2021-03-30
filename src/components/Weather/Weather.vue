<template>
    <div class="container">
        <!-- <template v-if="!isLoading"> -->
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
                                <div v-for="i in gotCities" :key="i.title+Math.random()" @click="getWeather(i.woeid)">
                                    {{i.title}}
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
                <img :src="'https://www.metaweather.com/static/img/weather/png/64/' + weather[0].weather_abbr + '.png'" alt="weather">

                <h1>{{weather[0].temp}}</h1>

                <p class="weatherState">{{weather[0].weather_name}}</p>

                <section class="footerSideBar">
                    <p>Today | {{date}}
                        <br /><br />

                        <span class="material-icons">location_on</span>
                        {{weather[0].title}}</p>
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
                        <img class="weatherIconSmall" :src="'https://www.metaweather.com/static/img/weather/png/' + i.weather_abbr + '.png'" alt="weather">
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
                    <p>created by <span style="color:#e94560;"><a href="https://www.pbielanin.pl" target="_blank">pbielanin</a></span></p>
                </section>
            </div>
        <!-- </template> -->

        <!-- <template v-else>
            <div class="loadingScreen">
                <div class="loader"></div>
            </div>
        </template> -->
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
                if(city == ""){
                    this.errorMessage = "No informations about this city";
                } else{
                    axios.get("https://www.metaweather.com/api/location/search/?query=" + city)
                        .then( ({data}) => {
                            this.gotCities = data;
                        })
                        .catch( (error) => {
                            this.errorMessage = "No informations about this city";
                        });
                }
            },

            getWeather(woeid){
                this.isCollectingData = true;
                this.modalOpen = false;
                axios.get("https://www.metaweather.com/api/location/" + woeid)
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
                        const wind_speed = (Math.round(this.currentWeather.consolidated_weather[i].wind_speed * 1.609344));
                        const visibility = (Math.round(this.currentWeather.consolidated_weather[i].wind_speed * 1.609344));
                        const air_pressure = this.currentWeather.consolidated_weather[i].air_pressure;
                        const temp = Math.round(this.currentWeather.consolidated_weather[i].the_temp);
                        const min_temp = Math.round(this.currentWeather.consolidated_weather[i].min_temp);
                        const max_temp= Math.round(this.currentWeather.consolidated_weather[i].max_temp);
                        this.weather.push({
                            "temp": temp + " °C",
                            "visibility": visibility + " km",
                            "wind_speed": wind_speed.toString() + " km/h",
                            "air_pressure": air_pressure + " hPa",
                            "min_temp": min_temp + " °C",
                            "max_temp": max_temp + " °C",
                            "humidity": this.currentWeather.consolidated_weather[i].humidity,
                            "weather_name": this.currentWeather.consolidated_weather[i].weather_state_name,
                            "weather_abbr": this.currentWeather.consolidated_weather[i].weather_state_abbr,
                            "date": this.currentWeather.consolidated_weather[i].applicable_date,
                            "title": this.currentWeather.title
                        })
                    } else{
                        const wind_speed = Math.round(this.currentWeather.consolidated_weather[i].wind_speed);
                        const visibility = Math.round(this.currentWeather.consolidated_weather[i].wind_speed);
                        const air_pressure = this.currentWeather.consolidated_weather[i].air_pressure;
                        const temp = Math.round(32 + (9/5) * this.currentWeather.consolidated_weather[i].the_temp);
                        const min_temp = Math.round(32 + (9/5) * this.currentWeather.consolidated_weather[i].min_temp);
                        const max_temp= Math.round(32 + (9/5) * this.currentWeather.consolidated_weather[i].max_temp);
                        this.weather.push({
                            "temp": temp + " °F",
                            "visibility": visibility + " miles",
                            "wind_speed": wind_speed.toString() + " mp/h",
                            "air_pressure": air_pressure + " mb",
                            "min_temp": min_temp + " °F",
                            "max_temp": max_temp + " °F",
                            "humidity": this.currentWeather.consolidated_weather[i].humidity,
                            "weather_name": this.currentWeather.consolidated_weather[i].weather_state_name,
                            "weather_abbr": this.currentWeather.consolidated_weather[i].weather_state_abbr,
                            "date": this.currentWeather.consolidated_weather[i].applicable_date,
                            "title": this.currentWeather.title
                        })
                    }
                };
            }
        },
        filters: {
            moment: function (date) {
                return moment(date).format("ddd, Do MMM");
            }
        },
        created(){
            currentWeather:
                axios.get("https://www.metaweather.com/api/location/523920")
                    .then( ({data}) => {
                        this.currentWeather = data;
                        this.isLoading = false;
                        this.europeConverter();
                    });

            date:
                this.date = moment().format("ddd, Do MMM");

        }
    }
</script>

<style lang="scss" scoped>

    .container{
        display: flex;
        flex-direction: row;
        flex-wrap: nowrap;
    }

    .SideBar{
        width: 25vw;
        height: auto;
        background-color: #16213e;
        color: whitesmoke;

        display: flex;
        flex-direction: column;
        flex-wrap: nowrap;
        justify-content: center;

        /* .SideBarBackground{
            position: absolute;
            top: 13%;
            left: 30px;
            width: 300px;
        } */

        button{
            padding: 15px;
            background-color: #e94560;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: color .5s;

            &:hover{
                color: #17172b;
            }

            &:focus{
                outline: none;
            }
        }

        img{
            width: 25%;
            z-index: 1;
        }

        h1{
            font-size: 54px;
        }

        .weatherState{
            font-size: 20px;
        }

        .footerSideBar{
            text-align: center;
            font-size: 14px;
            color: rgba(245, 245, 245, 0.479);
        }

        *{
            margin: auto;
        }

        .switchEuTablet, .switchUsTablet{
            display: none;
        }
    }

    .modalSearch{
        width: 22vw;
        height: 100vh;
        background-color: #131c35;
        color: whitesmoke;

        display: flex;
        flex-direction: column;
        flex-wrap: nowrap;
        justify-content: center;
        z-index: 10;
        position: absolute;
        margin: auto;
        top: 0;
        left: 0;

            .topModal{
                width: 100%;
                height: 20%;
                display: flex;
                flex-direction: column;
                justify-content: flex-start;

                span{
                    cursor: pointer;
                    width: 90%;
                    text-align: right;
                    transition: color .3s;

                    &:hover{
                        color: rgb(0, 20, 133);
                    }
                }

                input{
                    width: 90%;
                    padding: 10px;
                    border: none;
                    outline: none;
                }
            }

            .bottomModal{
                width: 100%;
                height: 80%;
                overflow: auto;

                div{
                    cursor: pointer;
                    list-style-type:none;
                    width: 90%;
                    padding: 15px;
                    background-color: #1a1a2e;

                    &:hover{
                        color: #5b5ba0;
                    }
                }

                p{
                    padding: 15px;
                }
            }
    }

    .weatherInfo{
        width: 75vw;
        height: 77.5vh;
        background-color: #17172b;
        color: white;
        padding: 100px;
        display: flex;
        flex-direction: column;
        justify-content: center;

        .switchEu, .switchUs{
            width: 75px;
            position: absolute;
            top: 40px;
            right: 70px;
            padding: 5px;
            border: none;
            border-radius: 5px;
            color: whitesmoke;
            background-color: #e94560;
            cursor: pointer;
            transition: color .5s;

            &:hover{
                color: #17172b;
            }

            &:focus{
                outline: none;
            }
        }

        .switchUs{
            background-color: #5b5ba0;
            transition: color .5s;

            &:hover{
                color: #e94560;
            }
        }


        .weatherForecast{
            width: 100%;
            height: 30%;
            display: flex;
            flex-direction: row;
            justify-content: center;

            div{
                height: 100%;
                width: 15%;
                background-color: #16213e;
                margin: 12px;
                display: flex;
                flex-direction: column;
                flex-wrap: nowrap;
                justify-content: center;

                *{
                    margin: auto;
                }

            }

            .weatherIconSmall{
                height: 60px;
                width: 60px;
            }
        }

        .weatherDetails{
            width: 100%;
            height: 70%;

            display: flex;
            flex-direction: row;
            flex-wrap: wrap;

            h3{
                width: 100%;
                margin-top: 60px;
                margin-left: 90px;
                margin-bottom: -30px;
            }

            .detailsTop{
                width: 100%;
                height: 40%;
                display: flex;
                flex-direction: row;
                justify-content: center;
                .square{
                    width: 40%;
                    height: 100%;
                    background-color: #16213e;
                    margin: 20px;

                    display: flex;
                    flex-direction: column;
                    flex-wrap: nowrap;
                    justify-content: center;

                    *{
                        margin: auto;
                    }
                    p:last-child{
                        margin-top: -20px;
                        font-size: 42px;
                        text-align: center;
                    }
                }
            }

            .detailsBottom{
                width: 100%;
                height: 30%;
                display: flex;
                flex-direction: row;
                justify-content: center;
                margin-top: 30px;

                .rectangle{
                    width: 40%;
                    height: 100%;
                    background-color: #16213e;
                    margin: 20px;

                    display: flex;
                    flex-direction: column;
                    flex-wrap: nowrap;
                    justify-content: center;

                    *{
                        margin: auto;
                    }
                    p:last-child{
                        margin-top: -20px;
                        font-size: 42px;
                        text-align: center;
                    }
                }
            }
        }

        .footer{
            position: absolute;
            bottom: 20px;
            display: flex;
            justify-content: center;
            width: 65%;
            color: gray;

            a{
                text-decoration: none;
                color: inherit;
            }
        }
    }

    .loadingScreen, .loadingDataScreen{
        width: 100vw;
        height: 100vh;
        background-color: rgba(0, 0, 0, 0.568);
        position: absolute;
        display: flex;
        justify-content: center;
        align-items: center;
        text-align: center;
        z-index: 999;

        .loader {
            border: 12px solid #1a1a2e;
            border-top: 12px solid #0f3460;
            border-radius: 50%;
            width: 60px;
            height: 60px;
            animation: spin 2s linear infinite;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    }

    .loadingScreen{
        background-color: #1a1a2e;
        .loader{
            border: 12px solid #22223b;
            border-top: 12px solid #0f3460;
        }
    }

    .list-enter,
    .list-leave-to {
        visibility: hidden;
        height: 0;
        margin: 0;
        padding: 0;
        opacity: 0;
    }

    .list-enter-active,
    .list-leave-active {
        transition: all 0.3s;
    }

    /* Laptops */
    @media (max-width: 1180px){
        .SideBar{
            width: 35vw;
        }

        .modalSearch{
            width: 25vw;
        }

        .weatherInfo{
            .weatherForecast{
                font-size: 14px;
                div{
                     width: 100%;
                     margin: 6px;
                }

                .weatherIconSmall{
                    height: 40px;
                    width: 40px;
                }
            }

            .weatherDetails{
                 h3{
                    width: 100%;
                    margin-top: 40px;
                    margin-left: 20px;
                    margin-bottom: -30px;
                }
                .detailsTop{
                    .square{
                        width: 100%;
                    }
                }
                .detailsBottom{
                    .rectangle{
                        width: 100%;
                    }
                }
            }
        }
    }

    /* Tablets */
    @media (max-width: 968px){
        .container{
            flex-direction: column;
            flex-wrap: nowrap;
        }

        .SideBar{
            width: auto;
            height: auto;

            button{
                margin-top: 50px;
            }

            img{
                margin-top: 50px;
                width: 15%;
            }

            h1{
                margin-top: 50px;
                font-size: 60px;
            }

            .weatherState{
                margin-top: 50px;
                font-size: 18px;
            }

            .footerSideBar{
                margin-top: 50px;
                font-size: 16px;
            }

            .switchEuTablet, .switchUsTablet{
                display: block;
                width: 95px;
                padding: 10px;
                border: none;
                border-radius: 5px;
                color: whitesmoke;
                background-color: #e94560;
                cursor: pointer;
                transition: color .5s;
                margin-bottom: 40px;

                &:hover{
                    color: #17172b;
                }

                &:focus{
                    outline: none;
                }
            }

            .switchUsTablet{
                background-color: #5b5ba0;
                transition: color .5s;

                &:hover{
                    color: #e94560;
                }
            }
        }


        .modalSearch{
            width: 100vw;
        }

        .weatherInfo{
            width: auto;
            height: 100vh;

            .switchUs, .switchEu{
                display: none;
            }
            .weatherForecast{
                font-size: 14px;
                div{
                     width: 100%;
                     margin: 6px;
                }

                .weatherIconSmall{
                    height: 40px;
                    width: 40px;
                }
            }

            .weatherDetails{
                 h3{
                    width: 100%;
                    margin-top: 40px;
                    margin-left: 20px;
                    margin-bottom: -30px;
                }
                .detailsTop{
                    .square{
                        width: 100%;
                    }
                }
                .detailsBottom{
                    .rectangle{
                        width: 100%;
                    }
                }
            }

            .footer{
                display: none;
            }
        }
    }

    @media (max-width: 710px){
        .weatherInfo{
            padding: 20px;
            .weatherForecast{
                font-size: 14px;
                width: 100%;
                div{
                     width: 100%;
                     height: auto;
                     margin: 6px;
                }

                .weatherIconSmall{
                    height: 40px;
                    width: 40px;
                }
            }

            .weatherDetails{
                 h3{
                    width: 100%;
                    margin-top: 40px;
                    margin-left: 20px;
                    margin-bottom: -30px;
                }
                .detailsTop{
                    .square{
                        width: 100%;

                        p:last-child{
                            margin-top: -20px;
                            font-size: 28px;
                        }
                    }
                }
                .detailsBottom{
                    .rectangle{
                        width: 100%;

                        p:last-child{
                            margin-top: -20px;
                            font-size: 28px;
                        }
                    }
                }
            }
        }
    }

    @media (max-width: 550px){
        .weatherInfo{
            padding: 20px;
            padding-bottom: 90px;
            .weatherForecast{
                display: flex;
                flex-direction: row;
                flex-wrap: wrap;
                font-size: 14px;
                width: 100%;
                div{
                    width: 100%;
                    height: auto;
                    margin: 6px;
                    display: flex;
                    flex-direction: row;
                    flex-wrap: nowrap;
                    justify-content: center;
                }

                .weatherIconSmall{
                    height: 40px;
                    width: 40px;
                }
            }

            .weatherDetails{
                 h3{
                    margin-top: 100px;
                }
                .detailsTop{
                    margin-top: 50px;
                    .square{
                        width: 100%;
                    }
                }
                .detailsBottom{

                    .rectangle{
                        width: 100%;
                    }
                }
            }
        }
    }

</style>
