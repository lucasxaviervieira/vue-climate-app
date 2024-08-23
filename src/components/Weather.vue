<template>
    <div>
        <div id="rain"></div>
        <div id="sky"></div>
        <div id="moon"></div>
        <div id="sun"></div>

        <div v-for="n in 8" :key="n" :id="'cloud' + n" class="cloud"></div>

        <button @click="toggleRain" class="btn btn-one btn-rain">Ativar/Desativar Chuva</button>
        <button @click="toggleDayNight" class="btn btn-one btn-day">Ativar/Desativar Noite</button>
        <button @click="started" id="toggleBtn" class="btn btn-one">Clique Aqui!</button>

        <div id="container">
            <input type=" text" v-model="city" placeholder="Cidade: " />
            <button @click="getLocation">Buscar</button>
            <div id="data-container">
                <div id="location"></div>
                <div id="precipitation_probability"></div>
                <div id="convective_precipitation"></div>
                <div id="time"></div>
                <div id="weather"></div>
            </div>
        </div>

        <div class="author">
            <a href="#">
                <img :src="logoImg" alt="Logo" />
            </a>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            isDay: true,
            city: "",
            rainInterval: null,
            containerVisible: true,
            weatherApiKey: "KuIW9skcDHPpQ7nv",
            logoImg: "/src/assets/logo-white.png"
        };
    },
    methods: {
        toggleDay(option) {
            const sky = document.getElementById("sky");
            const sun = document.getElementById("sun");
            const moon = document.getElementById("moon");
            const coulds = document.getElementsByClassName("cloud");
            const weather = document.getElementById("data-container");
            const button = document.querySelector("#container button");

            if (option !== "day") {
                moon.style.opacity = "1";
                sky.style.backgroundColor = "#000033";
                sun.style.transform = "translate(-80%, -160%)";
                moon.style.transform = "translate(-50%, -100%)";
                Array.from(coulds).forEach(cloud => cloud.style.backgroundColor = "#777");
                weather.style.color = "#fff";
                button.style.backgroundColor = "#333";
            } else {
                moon.style.opacity = "0";
                sky.style.backgroundColor = "#87CEEB";
                sun.style.transform = "translate(-50%, -160%)";
                button.style.backgroundColor = "#FFD700";
                Array.from(coulds).forEach(cloud => cloud.style.backgroundColor = "#fff");
                weather.style.color = "#000";
            }
        },
        started() {
            this.toggleDay("day");
            const sun = document.getElementById("sun");
            sun.style.transform = "translate(-50%, -90%)";
            this.containerVisible = true;
            const container = document.getElementById("container");
            const toggleBtn = document.getElementById("toggleBtn");
            container.style.width = "auto";
            container.style.height = "auto";
            container.style.overflow = "visible";
            container.classList.add("fadeIn");
            toggleBtn.style.display = "none";


        },
        toggleDayNight() {
            this.isDay = !this.isDay;
            this.toggleDay(this.isDay ? "day" : "night");
        },
        toggleRain() {
            // Implement rain toggle functionality
        },
        async fetchWeatherApi(lat, long, location) {
            await fetch(
                `https://my.meteoblue.com/packages/basic-1h_basic-day?apikey=${this.weatherApiKey}&lat=${lat}&lon=${long}&asl=275&format=json&forecast_days=1&history_days=1`
            )
                .then(response => response.json())
                .then(data => this.displayData(data, location))
                .catch(error => console.error("Erro ao carregar dados:", error));
        },
        async fetchGeoPositionApi() {
            await fetch(
                `https://geocoding-api.open-meteo.com/v1/search?name=${this.city}&count=10&language=en&format=json`
            )
                .then(response => response.json())
                .then(data => {
                    const latitude = data.results[0].latitude;
                    const longitude = data.results[0].longitude;
                    const location = `${data.results[0].name} - ${data.results[0].country}`;
                    this.fetchWeatherApi(latitude, longitude, location);
                })
                .catch(error => console.error("Erro ao carregar dados:", error));
        },
        calculateLocalTime(horaUtc, fusoHorario) {
            const hora = parseInt(horaUtc.split(" ")[1].split(":")[0]);
            const horaLocal = hora + fusoHorario;
            return horaLocal;
        },
        displayData(data, location) {
            const elementLocation = document.getElementById("location");
            const elementPrecipitation = document.getElementById(
                "precipitation_probability"
            );
            const elementWeather = document.getElementById("weather");

            const dataClimate = data["data_day"];
            const minClimate = `<h2>Temperatura Min. <span class="response">${dataClimate.temperature_min[1]}</span></h2>`;
            const currentClimate = `<h2>Temperatura Atual <span class="response">${dataClimate.temperature_instant[1]}</span></h2>`;
            const maxClimate = `<h2>Temperatura Max. <span class="response">${dataClimate.temperature_max[1]}</span></h2>`;

            const data_compiled = dataClimate.time[1].split("-");
            const get_data = data_compiled.join(" / ");

            const data1 = data["metadata"].modelrun_utc;
            const data2 = data["metadata"].utc_timeoffset;

            if (
                this.calculateLocalTime(data1, data2) >= 6 &&
                this.calculateLocalTime(data1, data2) <= 18
            ) {
                this.toggleDay("day");
            } else {
                this.toggleDay("night");
            }

            if (dataClimate.precipitation_probability[1] > 70) {
                this.startRain()
                elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Alta de chuva</p>`;
            } else if (dataClimate.precipitation_probability[1] >= 50) {
                this.startRain()
                elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Média de chuva</p>`;
            } else if (dataClimate.precipitation_probability[1] >= 40) {
                this.stopRain()
                elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Baixa de chuva</p>`;
            } else {
                this.stopRain()
                elementPrecipitation.innerHTML = `<p class="rain-legend">Dia Ensolarado</p>`;
            }

            elementLocation.innerHTML = `<div class="location-content"><p>${location}</p><p>${get_data}</p></div>`;
            elementWeather.innerHTML = minClimate + currentClimate + maxClimate;
        },
        getLocation() {
            if (this.city) {
                this.fetchGeoPositionApi();
            } else {
                console.log("error: field city is required");
            }
        },
        createRaindrop() {
            const raindrop = document.createElement("div");
            raindrop.className = "raindrop";
            raindrop.style.left = `${Math.random() * 100}vw`;
            raindrop.style.animationDuration = `${Math.random() * 2 + 1}s`;
            raindrop.style.opacity = `${Math.random()}`;
            document.getElementById("rain").appendChild(raindrop);

            setTimeout(() => {
                raindrop.remove();
            }, 1500);
        },
        startRain() {
            this.rainInterval = setInterval(this.createRaindrop, 50);
        },
        stopRain() {
            clearInterval(this.rainInterval);
            this.rainInterval = null;
        },
        toggleRain() {
            if (this.rainInterval) {
                this.stopRain();
            } else {
                this.startRain();
            }

        },
    },


    mounted() {
        this.toggleDay("day");
    },
};
</script>