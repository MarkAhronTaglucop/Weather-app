<template>
  <v-app class="custom-bg">
    <v-container>
      <v-row justify="center">
        <v-col cols="12" md="6">
          <v-card class="pa-4 elevation-12">
            <v-card-title>
              <h2 class="text-center">Weather Forecast</h2>
            </v-card-title>

            <v-card-text>
              <v-text-field
                v-model="city"
                label="Enter city"
                outlined
                @keyup.enter="fetchWeather"
              ></v-text-field>

              <v-btn color="primary" block @click="fetchWeather">
                Get Weather
              </v-btn>

              <v-alert v-if="error" type="error" class="mt-4">{{ error }}</v-alert>

              <!-- Loop through the cities array and display weather for each city -->
              <v-card
                v-for="(weather, index) in weatherData"
                :key="index"
                class="mt-4"
                :style="getWeatherCardStyle(weather)"
              >
                <v-card-title class="text-h5">{{ weather.name }}</v-card-title>
                <v-card-subtitle class="text-h6">
                  {{ capitalize(weather.weather[0].description) }}
                </v-card-subtitle>

                <v-card-text>
                  <v-row>
                    <v-col cols="6">
                      <p>
                        <strong>Temperature:</strong> {{ weather.main.temp }} °C
                      </p>
                      <p>
                        <strong>Humidity:</strong> {{ weather.main.humidity }}%
                      </p>
                      <p>
                        <strong>Pressure:</strong> {{ weather.main.pressure }} hPa
                      </p>
                      <p>
                        <strong>Wind Speed:</strong> {{ weather.wind.speed }} m/s
                      </p>
                      <p>
                        <strong>Sunrise:</strong> {{ formatTime(weather.sys.sunrise) }}
                      </p>
                      <p>
                        <strong>Sunset:</strong> {{ formatTime(weather.sys.sunset) }}
                      </p>
                    </v-col>
                  </v-row>
                </v-card-text>
              </v-card>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      city: "",
      weatherData: [], // Array to store weather data for up to 2 cities
      error: "",
    };
  },
  methods: {
    async fetchWeather() {
      if (this.city.trim() === "") {
        this.error = "Please enter a city name.";
        return;
      }
      try {
        const apiKey = "d384bf6937a8e37f888caae27c270cd2";
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=${apiKey}`
        );
        const data = await response.json();
        if (data.cod === 200) {
          this.addWeatherData(data);
          this.error = "";
        } else {
          this.error = "City not found.";
        }
      } catch (error) {
        this.error = "Unable to fetch weather data.";
      }
    },
    addWeatherData(weather) {
      // Add the weather data to the array
      if (!this.weatherData.some((w) => w.name === weather.name)) {
        this.weatherData.unshift(weather); // Add new data to the beginning of the array
        if (this.weatherData.length > 2) {
          this.weatherData.pop(); // Remove the oldest weather data if there are more than 2 entries
        }
      } else {
        this.error = "City already added.";
      }
      this.city = ""; // Clear the input field after fetching
    },
    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], {
        hour: "2-digit",
        minute: "2-digit",
      });
    },
    capitalize(text) {
      if (!text) return "";
      return text.charAt(0).toUpperCase() + text.slice(1);
    },
    getWeatherCardStyle(weather) {
      const condition = weather.weather[0].main.toLowerCase();
      switch (condition) {
        case "clear":
          return { backgroundColor: "#ffeb3b", color: "#000" }; // Sunny
        case "clouds":
          return { backgroundColor: "#90caf9", color: "#000" }; // Cloudy
        case "rain":
          return { backgroundColor: "#64b5f6", color: "#000" }; // Rainy
        case "snow":
          return { backgroundColor: "#bbdefb", color: "#000" }; // Snowy
        case "thunderstorm":
          return { backgroundColor: "#f44336", color: "#fff" }; // Thunderstorm
        default:
          return { backgroundColor: "#e0e0e0", color: "#000" }; // Default
      }
    },
  },
};
</script>

<style>
.custom-bg {
  background-color: #212121;
  min-height: 100vh;
}
body {
  font-family: "Roboto", sans-serif;
  background-color: rgb(99, 99, 179);
}

h2 {
  font-family: "Josefin Sans", sans-serif;
  color: #1976d2;
}

v-card-title {
  color: #424242;
}
</style>
