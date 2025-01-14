<script setup>
import { RouterLink, RouterView } from 'vue-router'
import { ref, computed } from 'vue';
import { onMounted } from 'vue'
import wmLogo from './assets/wm-logo.svg'
import wmLogo2 from './assets/wm-logo-2.svg'
const currentWeather = ref(null);
const location = ref(localStorage.getItem('weatherLocation') || '');
//removed isInitialValue to make changes to input field better
const isInitialValue = ref(!localStorage.getItem('weatherLocation'));
const inputError = ref('');
const locationInput = ref(null);

const getLocation = () => {
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(async (position) => {
      const { latitude, longitude } = position.coords;
      location.value = `${latitude},${longitude}`;
      await fetchWeather();
    }, (error) => {
      console.error("Error getting location:", error);
    });
  } else {
    console.log("Geolocation is not supported by this browser.");
  }
};

onMounted(async() => {
  if (location.value) {
    await fetchWeather();
    setTimeout(scrollToCurrentHour, 100);
  }
 /*   else {
    // Prompt for location if we don't have a saved one
    getLocation();
  } */
});

const fetchWeather = async () => {
  if (!location.value) {
    inputError.value = 'Please enter a ZIP Code';
    return;
  }
  //clear error
  inputError.value = ''; 
  try {
    const response = await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${import.meta.env.VITE_WEATHER_API_KEY}&q=${location.value}&days=2&aqi=no&alerts=no`);
    const data = await response.json();
    currentWeather.value = data;
    localStorage.setItem('weatherLocation', location.value);
    setTimeout(scrollToCurrentHour, 100);
    locationInput.value?.blur();
  } catch (error) {
    console.error('Error fetching data:', error);
    inputError.value = 'Unable to fetch weather data';
  }
}
//removed isInitialValue to just clear input value below
/* const handleFocus = () => {
  if (isInitialValue.value) {
    location.value = '';
    isInitialValue.value = false;
  }
} */
const handleFocus = () => {
  location.value = '';
}

const getTimeClass = computed(() => {
  return currentWeather.value?.current.is_day ? 'day-mode' : 'night-mode'
})

const currentHourIndex = computed(() => {
  if (!currentWeather.value) return 0;
  const now = new Date();
  return currentWeather.value.forecast.forecastday[0].hour.findIndex(hour => {
    const hourTime = new Date(hour.time);
    return hourTime.getHours() === now.getHours();
  });
});

const scrollToCurrentHour = () => {
  const scrollContainer = document.querySelector('.forecast-scroll');
  const hourElements = document.querySelectorAll('.forecast-item');
  if (scrollContainer && hourElements[currentHourIndex.value]) {
    const hourElement = hourElements[currentHourIndex.value];
    const scrollOffset = hourElement.offsetLeft - (scrollContainer.offsetWidth / 2) + (hourElement.offsetWidth / 2);
    scrollContainer.scrollTo({ left: scrollOffset, behavior: 'smooth' });
  }
};
</script>

<template>
  <div :class="[getTimeClass, 'weather-info app-container']">
   <!--  <header>
      <div class="wrapper">
        <nav>
          <RouterLink to="/">Home</RouterLink>
          <RouterLink to="/about">About</RouterLink>
        </nav>
      </div>
    </header> -->
<!--     <img :src="wmLogo" alt="logo" height="131" width="225" class="block mx-auto mt-4" />
     
 -->    
 <img :src="wmLogo2" alt="logo" height="100" width="175" class="block mx-auto mt-4" />
 <div class="flex flex-row justify-center items-center p-4">
      <form action="" @submit.prevent="fetchWeather" class="flex flex-col items-center">
        <div class="input-group">
          <input 
            ref="locationInput"
            type="text" 
            v-model="location" 
            placeholder="Enter a ZIP Code" 
            @focus="handleFocus"
            class="border border-gray-300 rounded-md p-2 text-black" 
            :class="{ 'border-red-500': inputError }"
        />
        <button type="submit" class="bg-blue-500 text-white rounded-md p-2 ml-2">Search</button>
        </div>
        <!-- Error message -->
        <div v-if="inputError" class="text-red-500 text-sm mt-1">
          {{ inputError }}
        </div>
        <div class="or block mt-2 relative w-full text-center">
          <span :class="[getTimeClass, 'weather-info relative z-10 text-white px-2 ']">or</span>
        </div>
        <button 
          type="button" 
          @click="getLocation" 
          class="bg-blue-500 text-white rounded-md p-2 mt-2 ml-2 flex flex-row items-center"
        >
        <svg class="w-5 h-5" fill="#fff" version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" 
	 width="800px" height="800px" viewBox="0 0 395.71 395.71"
	 xml:space="preserve">
<g>
	<path d="M197.849,0C122.131,0,60.531,61.609,60.531,137.329c0,72.887,124.591,243.177,129.896,250.388l4.951,6.738
		c0.579,0.792,1.501,1.255,2.471,1.255c0.985,0,1.901-0.463,2.486-1.255l4.948-6.738c5.308-7.211,129.896-177.501,129.896-250.388
		C335.179,61.609,273.569,0,197.849,0z M197.849,88.138c27.13,0,49.191,22.062,49.191,49.191c0,27.115-22.062,49.191-49.191,49.191
		c-27.114,0-49.191-22.076-49.191-49.191C148.658,110.2,170.734,88.138,197.849,88.138z"/>
</g>
</svg> <span class="ml-2">Use My Location</span>
        </button>
      </form>
    </div>
    <div v-if="currentWeather" :class="[getTimeClass, 'weather-info p-4 overflow-x-hidden']">
      <div class="max-w-4xl mx-auto w-full">
        <div class="current-weather-container">
          <h2 class="text-3xl font-bold text-center">Weather in <div class="italic">{{ currentWeather.location.name }}, {{ currentWeather.location.region }}</div></h2>
          <div class="mt-2">
            <div class="flex flex-row text-2xl w-full justify-center items-center">
            It's {{ currentWeather.current.temp_f }}°F and {{ currentWeather.current.condition.text }}
            <img :src="currentWeather.current.condition.icon" alt="weather icon" />
          </div>
          <p class="text-sm text-center">Feels like: {{ currentWeather.current.feelslike_f }}°F</p>
        <!--   <p class="text-lg">Humidity: {{ currentWeather.current.humidity }}%</p> -->
        </div>
        </div>
        <div class="forecast-container mt-4">
          <div class="forecast-scroll">
            <div v-for="(hour, index) in currentWeather.forecast.forecastday[0].hour" 
                 :key="hour.time"
                 :class="['forecast-item', { 'current-hour': index === currentHourIndex }]">
              <p class="time">{{ new Date(hour.time).toLocaleTimeString('en-US', { hour: 'numeric' }) }}</p>
              <img :src="hour.condition.icon" :alt="hour.condition.text" class="w-10 h-10 mx-auto" />
              <p class="temp">{{ Math.round(hour.temp_f) }}°F</p>
            </div>
          </div>
        </div>

        <div class="mt-6 p-4 bg-white/10 rounded-lg backdrop-blur-sm">
          <h3 class="text-xl font-semibold">Tomorrow's Forecast</h3>
          <div class="flex items-center gap-2">
            <img 
              :src="currentWeather.forecast.forecastday[1].day.condition.icon" 
              :alt="currentWeather.forecast.forecastday[1].day.condition.text"
              class="w-10 h-10"
            />
            <div>
              <p>High: {{ Math.round(currentWeather.forecast.forecastday[1].day.maxtemp_f) }}°F</p>
              <p>Low: {{ Math.round(currentWeather.forecast.forecastday[1].day.mintemp_f) }}°F</p>
              <p>{{ currentWeather.forecast.forecastday[1].day.condition.text }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <RouterView />
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  max-width: 100vw;
  overflow-x: hidden;
  position: relative;
  min-height: 100dvh;
}
.current-weather-container {
    background-color: rgba(255, 255, 255, .5);
    padding: 1rem;
    border-radius: 1rem;
} 
.or {
    position: relative;
}
.or::after {
    content: " ";
    position: absolute;
    top: 50%;
    left: 0;
    right: 0;
    transform: translateY(-50%);
    width: 100%;
    height: 1px;
    background-color: #fff;
}
header {

}

.logo {

}

nav {

}

nav a.router-link-exact-active {
}

nav a.router-link-exact-active:hover {
}

nav a {
 
}

nav a:first-of-type {

}

@media (min-width: 1024px) {
  header {
   
  }

  .logo {
  }

  header .wrapper {

  }

  nav {
   
  }
}

.day-mode {
  background-color: #87CEEB; /* Light blue sky color */
  color: #333;
}

.night-mode {
  background-color: #1a237e; /* Dark blue night color */
  color: #fff;
}

.forecast-container {
  width: 100%;
  padding: 1rem 0;
  overflow: hidden;
  position: relative;
}

.forecast-scroll {
  display: flex;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  gap: 1rem;
  padding: 0.5rem;
  -webkit-overflow-scrolling: touch;
  width: 100%;
  margin: 0 auto;
  position: relative;
}

.forecast-scroll::-webkit-scrollbar {
  height: 8px;
}

.forecast-scroll::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
}

.forecast-scroll::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.3);
  border-radius: 4px;
}

.forecast-item {
  flex: 0 0 auto;
  scroll-snap-align: start;
  min-width: 80px;
  padding: 0.5rem;
  text-align: center;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  backdrop-filter: blur(4px);
}

.forecast-item .time {
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.forecast-item .temp {
  font-size: 1.1rem;
  font-weight: bold;
}

.forecast-item.current-hour {
  background: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.5);
}
</style>
