<template>
  <main>
    <Configs></Configs>
    <Clouds></Clouds>
    <section>
      <button @click="toggleRain" class="btn btn-one btn-rain show-in-desktop">Ativar/Desativar Chuva</button>
      <button @click="toggleDayNight" class="btn btn-one btn-day show-in-desktop">Ativar/Desativar Noite</button>
      <button @click="started" id="toggleBtn" class="btn btn-one">Clique Aqui!</button>
    </section>
    <section id="container">
      <input type=" text" v-model="city" placeholder="Cidade: " />
      <button @click="getLocation">Buscar</button>
      <Results></Results>
    </section>
    <Author></Author>
  </main>
</template>

<script setup>
import { ref, onMounted } from "vue";
import Clouds from "./components/Clouds.vue";
import Configs from "./components/Configs.vue";
import Author from "./components/Author.vue";
import Results from "./components/Results.vue";

const isDay = ref(true);
const city = ref("");
const rainInterval = ref(null);
const containerVisible = ref(true);
const weatherApiKey = "KuIW9skcDHPpQ7nv";

const toggleDay = (option) => {
  const sky = document.getElementById("sky");
  const sun = document.getElementById("sun");
  const moon = document.getElementById("moon");
  const clouds = document.getElementsByClassName("cloud");
  const weather = document.getElementById("data-container");
  const button = document.querySelector("#container button");

  if (option !== "day") {
    moon.style.opacity = "1";
    sky.style.backgroundColor = "#000033";
    sun.style.transform = "translate(-80%, -160%)";
    moon.style.transform = "translate(-50%, -100%)";
    Array.from(clouds).forEach(cloud => cloud.style.backgroundColor = "#777");
    weather.style.color = "#fff";
    button.style.backgroundColor = "#333";
  } else {
    moon.style.opacity = "0";
    sky.style.backgroundColor = "#87CEEB";
    sun.style.transform = "translate(-50%, -160%)";
    button.style.backgroundColor = "#FFD700";
    Array.from(clouds).forEach(cloud => cloud.style.backgroundColor = "#fff");
    weather.style.color = "#000";
  }
};

const started = () => {
  toggleDay("day");
  const sun = document.getElementById("sun");
  sun.style.transform = "translate(-50%, -90%)";
  containerVisible.value = true;
  const container = document.getElementById("container");
  const toggleBtn = document.getElementById("toggleBtn");
  container.style.width = "auto";
  container.style.height = "auto";
  container.style.overflow = "visible";
  container.classList.add("fadeIn");
  toggleBtn.style.display = "none";
};

const toggleDayNight = () => {
  isDay.value = !isDay.value;
  toggleDay(isDay.value ? "day" : "night");
};

const fetchWeatherApi = async (lat, long, location) => {
  try {
    const response = await fetch(
      `https://my.meteoblue.com/packages/basic-1h_basic-day?apikey=${weatherApiKey}&lat=${lat}&lon=${long}&asl=275&format=json&forecast_days=1&history_days=1`
    );
    const data = await response.json();
    displayData(data, location);
  } catch (error) {
    console.error("Erro ao carregar dados:", error);
  }
};

const fetchGeoPositionApi = async () => {
  try {
    const response = await fetch(
      `https://geocoding-api.open-meteo.com/v1/search?name=${city.value}&count=10&language=en&format=json`
    );
    const data = await response.json();
    const latitude = data.results[0].latitude;
    const longitude = data.results[0].longitude;
    const location = `${data.results[0].name} - ${data.results[0].country}`;
    fetchWeatherApi(latitude, longitude, location);
  } catch (error) {
    console.error("Erro ao carregar dados:", error);
  }
};

const calculateLocalTime = (horaUtc, fusoHorario) => {
  const hora = parseInt(horaUtc.split(" ")[1].split(":")[0]);
  const horaLocal = hora + fusoHorario;
  return horaLocal;
};

const displayData = (data, location) => {
  const elementLocation = document.getElementById("location");
  const elementPrecipitation = document.getElementById("precipitation_probability");
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
    calculateLocalTime(data1, data2) >= 6 &&
    calculateLocalTime(data1, data2) <= 18
  ) {
    toggleDay("day");
  } else {
    toggleDay("night");
  }

  if (dataClimate.precipitation_probability[1] > 70) {
    startRain();
    elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Alta de chuva</p>`;
  } else if (dataClimate.precipitation_probability[1] >= 50) {
    startRain();
    elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Média de chuva</p>`;
  } else if (dataClimate.precipitation_probability[1] >= 40) {
    stopRain();
    elementPrecipitation.innerHTML = `<p class="rain-legend">Chance Baixa de chuva</p>`;
  } else {
    stopRain();
    elementPrecipitation.innerHTML = `<p class="rain-legend">Dia Ensolarado</p>`;
  }

  elementLocation.innerHTML = `<div class="location-content"><p>${location}</p><p>${get_data}</p></div>`;
  elementWeather.innerHTML = minClimate + currentClimate + maxClimate;
};

const getLocation = () => {
  if (city.value) {
    fetchGeoPositionApi();
  } else {
    console.log("error: field city is required");
  }
};

const createRaindrop = () => {
  const raindrop = document.createElement("div");
  raindrop.className = "raindrop";
  raindrop.style.left = `${Math.random() * 100}vw`;
  raindrop.style.animationDuration = `${Math.random() * 2 + 1}s`;
  raindrop.style.opacity = `${Math.random()}`;
  document.getElementById("rain").appendChild(raindrop);

  setTimeout(() => {
    raindrop.remove();
  }, 1500);
};

const startRain = () => {
  rainInterval.value = setInterval(createRaindrop, 50);
};

const stopRain = () => {
  clearInterval(rainInterval.value);
  rainInterval.value = null;
};

const toggleRain = () => {
  if (rainInterval.value) {
    stopRain();
  } else {
    startRain();
  }
};

onMounted(() => {
  toggleDay("day");
});
</script>
