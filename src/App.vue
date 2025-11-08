<template>
  <div
    id="app"
    :class="
      typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm' : ''
    "
  >
    <main>
      <div class="search-box">
        <input
          type="text"
          class="search-bar"
          placeholder="Search..."
          v-model="query"
          @input="fetchSuggestions"
          @keypress.enter="searchSelectedCity"
        />
        <ul v-if="suggestions.length" class="suggestions">
          <li
            v-for="(city, index) in suggestions"
            :key="index"
            @click="selectCity(city)"
          >
            {{ city.name }}, {{ city.country }}
          </li>
        </ul>
      </div>

      <div
        class="weather-wrap"
        :class="{ animate: showWeather }"
        v-if="typeof weather.main != 'undefined'"
      >
        <div class="location-box">
          <div class="location">
            {{ weather.name }}, {{ weather.sys.country }}
          </div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°c</div>
          <div class="weather">{{ weather.weather[0].main }}</div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      api_key: "16ae4da87158f2276814c9d1a14a9612",
      url_base: "https://api.openweathermap.org/data/2.5/",
      geo_base: "http://api.openweathermap.org/geo/1.0/direct",
      query: "",
      weather: {},
      showWeather: false,
      suggestions: [],
      selectedCity: null,
    };
  },
  methods: {
    fetchSuggestions() {
      if (this.query.length < 2) {
        this.suggestions = [];
        return;
      }

      fetch(`${this.geo_base}?q=${this.query}&limit=5&appid=${this.api_key}`)
        .then((res) => res.json())
        .then((data) => {
          this.suggestions = data.map((city) => ({
            name: city.name,
            country: city.country,
            lat: city.lat,
            lon: city.lon,
          }));
        });
    },

    selectCity(city) {
      this.query = `${city.name}, ${city.country}`;
      this.selectedCity = city;
      this.suggestions = [];
      this.fetchWeatherByCoords(city.lat, city.lon);
    },

    searchSelectedCity() {
      if (this.selectedCity) {
        this.fetchWeatherByCoords(this.selectedCity.lat, this.selectedCity.lon);
      } else if (this.query.trim() !== "") {
        this.fetchWeatherByName(this.query.trim());
      }
    },

    fetchWeatherByCoords(lat, lon) {
      this.showWeather = false;
      fetch(
        `${this.url_base}weather?lat=${lat}&lon=${lon}&units=metric&appid=${this.api_key}`
      )
        .then((res) => res.json())
        .then((data) => {
          this.weather = data;
          this.$nextTick(() => {
            this.showWeather = true;
          });
        });
    },

    fetchWeatherByName(name) {
      this.showWeather = false;
      fetch(
        `${this.url_base}weather?q=${name}&units=metric&appid=${this.api_key}`
      )
        .then((res) => res.json())
        .then((data) => {
          this.weather = data;
          this.$nextTick(() => {
            this.showWeather = true;
          });
        });
    },

    dateBuilder() {
      let d = new Date();
      let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      return `${days[d.getDay()]} ${d.getDate()} ${
        months[d.getMonth()]
      } ${d.getFullYear()}`;
    },
  },
};
</script>

<style>
/* إضافة تصميم للاقتراحات */
.suggestions {
  list-style: none;
  background: rgba(255, 255, 255, 0.9);
  max-height: 200px;
  overflow-y: auto;
  margin-top: 5px;
  border-radius: 8px;
  padding: 0;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}
.suggestions li {
  padding: 10px;
  cursor: pointer;
}
.suggestions li:hover {
  background: rgba(0, 0, 0, 0.1);
}
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: "montserrat", sans-serif;
}

#app {
  background-image: url("./assets/cold-bg.jpg");
  background-size: cover;
  background-position: center;
  transition: 0.4s;
}

#app.warm {
  background-image: url("./assets/warm-bg.jpg");
}

main {
  min-height: 100vh;
  padding: 25px;
  background-image: linear-gradient(
    to bottom,
    rgba(0, 0, 0, 0.25),
    rgba(0, 0, 0, 0.75)
  );
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
  position: relative;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0 0 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.suggestions {
  list-style: none;
  padding: 0;
  margin: 5px 0 0 0;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  max-height: 150px;
  overflow-y: auto;
  position: absolute;
  width: 100%;
  z-index: 10;
}

.suggestions li {
  padding: 8px 12px;
  cursor: pointer;
  transition: background 0.2s;
}

.suggestions li:hover {
  background: rgba(0, 0, 0, 0.1);
}

.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}

.location-box .date {
  color: #fff;
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
  color: #fff;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-wrap {
  opacity: 0;
  transform: translateY(30px) scale(0.95);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.weather-wrap.animate {
  opacity: 1;
  transform: translateY(0) scale(1);
}

@media (min-width: 426px) {
  body {
    background-color: #0d1729;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }

  #app {
    width: 425px;
    height: 956px;
    border-radius: 20px;
    overflow: hidden;
  }

  main {
    min-height: 100%;
    border-radius: 20px;
  }
}
</style>
