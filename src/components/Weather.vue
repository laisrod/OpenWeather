<template>
  <div>
    <input
    v-model="city"
    placeholder="Nome da cidade"
    @keyup.enter="fetchWeatherForecast"
    />
    <button @click="fetchWeatherForecast">Buscar</button>
  </div>

  <div v-if="forecast" class="forecast-info">
    <section>
      <h2>Dados da cidade:</h2>
      <p>Nome: {{ forecast.city.name }}</p>
      <p>País: {{ forecast.city.country }}</p>
      <p>Latitude: {{ forecast.city.coord.lat }}</p>
      <p>Longitude: {{ forecast.city.coord.lon }}</p>
      <p>População: {{ forecast.city.population }}</p>
      <p>Fuso Horário: {{ forecast.city.timezone }}</p>
      <p>Nascer do Sol: {{ formatDateTime(forecast.city.sunrise) }}</p>
      <p>Pôr do Sol: {{ formatDateTime(forecast.city.sunset) }}</p>
    </section>

    <section>
      <h2>Previsão:</h2>
      <div v-for="item in forecast.list" :key="item.dt" class="forecast-item">
        <p>Data e Hora: {{ formatDateTime(item.dt) }}</p>
        <p>Temperatura Atual: {{ item.main.temp }} °C</p>
        <p>Sensação Térmica: {{ item.main.feels_like }} °C</p>
        <p>Temperatura Mínima: {{ item.main.temp_min }} °C</p>
        <p>Temperatura Máxima: {{ item.main.temp_max }} °C</p>
        <p>Pressão Atmosférica: {{ item.main.pressure }} hPa</p>
        <p>Umidade: {{ item.main.humidity }}%</p>
        <p>Descrição do Clima: {{ item.weather[0].description }}</p>
        <img :src="getWeatherIcon(item.weather[0].icon)" :alt="item.weather[0].description">
        <p>Nuvens: {{ item.clouds.all }}%</p>
        <p>Velocidade do Vento: {{ item.wind.speed }} m/s</p>
        <p>Direção do Vento: {{ item.wind.deg }}°</p>
        <p>Precipitação (3h): {{ item.rain?.['3h'] || 'Sem Dados' }} mm</p>
        <p>Visibilidade: {{ item.visibility }} metros</p>
        <p>Probabilidade de Precipitação: {{ item.pop * 100 }}%</p>
        <hr>
      </div>
    </section>
  </div>
  
  <div v-else-if="errorMessage" class="errorMessage">
    <p>{{ errorMessage }}</p>
  </div>
</template>

<script>
  import axios from "axios";

  export default {
    data() {
      return {
        city: "",
        forecast: null,
        errorMessage: null
      };
    },
    methods: {
      async fetchWeatherForecast() {
        if (!this.city.trim()) {
          this.forecast = null;
          this.errorMessage = "Digite o nome da cidade para obter a previsão.";
          return;
        }

        const apiKey = import.meta.env.VITE_OPENWEATHER_API_KEY;
        const url = `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&appid=${apiKey}&units=metric&lang=pt_br`;

        try {
          const response = await axios.get(url);
          if (response.status === 200 && response.data.cod === "200") {
            this.forecast = response.data;
            this.errorMessage = null;
          }
        } catch (error) {
          this.forecast = null;

          if (error.response) {
            const apiMessage = error.response.data.message;
            this.errorMessage = apiMessage === "city not found"
              ? "Cidade não encontrada. Por favor, verifique o nome da cidade e tente novamente."
              : apiMessage || "Erro ao buscar cidade.";
          } else if (error.request) {
            this.errorMessage = "Nenhuma resposta recebida. Verifique a conexão.";
          } else {
            this.errorMessage = "Erro inesperado ao buscar dados";
          }
          console.error("Erro ao buscar API:", error);
        }
      },

      formatDateTime(dateTime) {
        return new Date(dateTime * 1000).toLocaleString("pt-BR", {
          dateStyle: "short",
          timeStyle: "short",
        });
      },

      getWeatherIcon(icon) {
        return `https://openweathermap.org/img/wn/${icon}.png`;
      },
    },
  }
</script>

<style scoped>
  .forecast-info {
    margin-top: 20px;
  }

  section {
    margin-bottom: 30px;
  }

  h2 {
    border-bottom: 2px solid #007bff;
    padding-bottom: 10px;
  }

  .forecast-item {
    background-color: #2c2c2e;
    color: #ffffff;
    padding: 15px;
    margin: 10px 0;
    border-radius: 8px;
    border-left: 4px solid #007bff;
  }

  .forecast-item p {
    margin: 5px 0;
  }

  .forecast-item img {
    width: 50px;
    height: 50px;
    vertical-align: middle;
  }

  .errorMessage {
    color: #ff4d4d;
    background-color: #442a2a;
    border: 1px solid #ff4d4d;
    padding: 15px;
    border-radius: 4px;
    margin-top: 20px;
  }

  hr {
    border: none;
    border-top: 1px solid #444;
    margin: 15px 0;
  }

  input {
    padding: 10px;
    font-size: 16px;
    border: 1px solid #555;
    border-radius: 4px;
    margin-right: 10px;
    background-color: #333;
    color: #fff;
  }

  button {
    padding: 10px 20px;
    font-size: 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }
</style>