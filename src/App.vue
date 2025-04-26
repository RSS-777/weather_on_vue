<script setup lang="ts">
import { ref } from "vue";

interface IWeatherData {
  main: {
    temp: number;
    feels_like: number;
    pressure: number;
    humidity: number;
  };
  name: string;
  sys: {
    type: number;
    id: number;
    country: string;
    sunrise: number;
    sunset: number;
  };
  timezone: number;
  visibility: number;
  weather: {
    description: string;
    icon: string;
  }[];
  wind: {
    speed: number;
    deg: number;
    gust: number;
  };
}

const city = ref<string>('');
const data = ref<IWeatherData | null>(null);
const errorMessage = ref<string | null>(null)

const getWeather = async () => {
  errorMessage.value = null
  data.value = null

  if (!city.value) return

  if (!/^[a-zA-Zа-яА-ЯёЁіІїЇєЄ\s]+$/u.test(city.value)) {
    errorMessage.value = 'Назва населеного пункту може містити лише літери та пробіли.';
    return;
  }

  try {
    const response = await fetch(`/api/getData?city=${city.value}`)
    const result = await response.json();

    if (response.ok) {
      if (result && result.sys && result.weather) {
        data.value = result;
      } else {
        errorMessage.value = 'Не вдалося знайти дані для вказаного міста.';
      }
    } else {
      errorMessage.value = result.message || 'Сталася помилка при отриманні даних.';
    }
  } catch (error: any) {
    errorMessage.value = error.message
  }
}
</script>

<template>
  <div class="wrapper">
    <main>
      <h1>Додаток погоди</h1>
      <p>Дізнайтесь погоду у вашому населеному пункті.</p>
      <div class="block-input">
        <input type="text" placeholder="Введіть населений пункт" aria-label="Назва населеного пункту" v-model="city" />
        <button type="button" :disabled="city === ''" @click="getWeather">Отримати погоду</button>
      </div>
      <div class="weather" v-if="data && !errorMessage">
        <div class="block-info">
          <span class="block-info__item1">{{ data.sys.country }}</span>
          <span class="block-info__item2">{{ data.name }}</span>
          <span class="block-info__item3">{{ data.weather[0].description.charAt(0).toUpperCase() +
            data.weather[0].description.slice(1) }}</span>
          <span class="block-info__item4"> Таймзона: <br />{{ data.timezone > 0 ? `+${data.timezone / 3600}` :
            `${data.timezone / 3600}` }}</span>
          <img class="block-info__item-center" :src="`https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`"
            alt="Image weather">
        </div>
        <div class="block-wind">
          <div class="arrow" :style="{ transform: 'translate(-50%, -50%) rotate(' + data.wind.deg + 'deg)' }">&#8594;
          </div>
          <span class="s">Пд</span>
          <span class="n">Пн</span>
          <span class="e">Сх</span>
          <span class="w">Зх</span>
          <span class="speed">S: {{data.wind.speed ? (data.wind.speed.toFixed(1) + ' м/с') : 'N/A' }}</span>
          <span class="gust">G: {{  data.wind.gust ? (data.wind.gust.toFixed(1) + ' м/с') : 'N/A' }}</span>
        </div>
        <div class="block-details">
          <span class="block-details__item1">Вологість:<br /> {{ data.main.humidity }}</span>
          <span class="block-details__item2">Тиск: <br /> {{ data.main.pressure + ' hPa' }}</span>
          <span class="block-details__item3">Відчув. : {{ data.main.feels_like }}</span>
          <span class="block-details__item4">Видимість: {{ data.visibility + ' m' }}</span>
          <span class="block-details__item-center">{{ data.main.temp ? (data.main.temp.toFixed(1) + '&deg;C') : 'N/A' }}</span>
        </div>
      </div>
      <div class="block-error">
        <p v-show="errorMessage">{{ errorMessage }}</p>
      </div>
    </main>
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 10px;
}

.wrapper main {
  position: relative;
  padding: 20px 10px;
  max-width: 600px;
  border-radius: 15px;
  width: 100%;
  min-height: 350px;
  background-color: rgb(3, 5, 34);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.wrapper main h1 {
  color: white;
}

.wrapper main>p {
  color: white;
  padding: 20px;
}

.block-input input {
  background-color: transparent;
  border-width: 0 0 2px 0;
  outline: none;
  color: white;
}

.block-input button {
  padding: 2px 5px;
  margin-left: 10px;
  border-radius: 8px;
  background-color: rgb(215, 189, 44);
  color: white;
  border: none;
  box-shadow: 2px 2px 8px 0 rgb(5, 2, 89);
  transition: transform 500ms ease;
  cursor: pointer;
}

.block-input button:hover {
  transform: scale(1.05);
}

.block-input button:disabled {
  background-color: rgb(135, 135, 134);
  transform: scale(1);
  cursor: auto;
}

.block-error {
  position: absolute;
  bottom: 0;
  color: brown;
  padding: 10px;
  width: 100%;
  text-align: center;
}

.weather {
  position: relative;
  color: white;
  display: flex;
  gap: 10px;
  justify-content: space-evenly;
  margin: 20px 10px;
  width: 100%;
  font-size: 14px;
}

.weather .block-info {
  position: relative;
  display: grid;
  grid-template: repeat(2, 1fr)/ repeat(2, 1fr);
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;
  border: 2px solid rgb(8, 122, 203);
  border-radius: 50%;
  overflow: hidden;
  text-align: center;
  box-shadow: 0 0 5px 0 rgb(254, 254, 136);
}

.block-info .block-info__item1 {
  padding: 20px 10px 0 20px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 2px solid rgb(8, 122, 203);
  border-right: 2px solid rgb(8, 122, 203);
}

.block-info .block-info__item2 {
  padding: 20px 20px 0 10px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 2px solid rgb(8, 122, 203);
}

.block-info .block-info__item3 {
  padding: 0 10px 20px 20px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-right: 2px solid rgb(8, 122, 203);
}

.block-info .block-info__item4 {
  padding: 0 20px 20px 10px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.block-info .block-info__item-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 50px;
  height: 50px;
  border: 2px solid rgb(8, 122, 203);
  background-color: #030522;
  border-radius: 50%;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
}

.weather .block-details {
  position: relative;
  display: grid;
  grid-template: repeat(2, 1fr)/ repeat(2, 1fr);
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;
  border: 2px solid rgb(8, 122, 203);
  border-radius: 50%;
  overflow: hidden;
  text-align: center;
  box-shadow: 0 0 5px 0 rgb(254, 254, 136);
}

.block-details .block-details__item1 {
  padding: 20px 10px 0 20px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 2px solid rgb(8, 122, 203);
  border-right: 2px solid rgb(8, 122, 203);
}

.block-details .block-details__item2 {
  padding: 20px 20px 0 10px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-bottom: 2px solid rgb(8, 122, 203);
}

.block-details .block-details__item3 {
  padding: 0 10px 20px 20px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  border-right: 2px solid rgb(8, 122, 203);
}

.block-details .block-details__item4 {
  padding: 0 20px 20px 10px;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.block-details .block-details__item-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 50px;
  height: 50px;
  border: 2px solid rgb(8, 122, 203);
  background-color: #030522;
  border-radius: 50%;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
}

.block-wind {
  position: relative;
  width: 70px;
  height: 70px;
  border: 2px solid rgb(8, 122, 203);
  border-radius: 50%;
  box-shadow: 0 0 5px 0 rgb(254, 254, 136);
}

.block-wind .arrow {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  color: red;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  font-size: 14px;
  border: 1px solid transparent;
  z-index: 2;
}

.block-wind .arrow::before {
  content: '';
  position: absolute;
  top: 15px;
  left: 15px;
  transform: translate(-50%, -50%);
  width: 5px;
  height: 5px;
  background-color: rgb(8, 122, 203);
  border-radius: 50%;
}

.block-wind .s {
  font-size: 8px;
  position: absolute;
  top: 50%;
  right: 2px;
  transform: translateY(-50%);
}

.block-wind .n {
  font-size: 8px;
  position: absolute;
  top: 50%;
  left: 2px;
  transform: translateY(-50%);
}

.block-wind .e {
  font-size: 8px;
  position: absolute;
  top: 2px;
  left: 50%;
  transform: translateX(-50%);
}

.block-wind .w {
  font-size: 8px;
  position: absolute;
  bottom: 2px;
  left: 50%;
  transform: translateX(-50%);
}

.block-wind .speed {
  font-size: 7px;
  position: absolute;
  top: 16px;
  left: 50%;
  transform: translateX(-50%);
  color: #D7BD2C;
}

.block-wind .gust {
  font-size: 7px;
  position: absolute;
  bottom: 16px;
  left: 50%;
  transform: translateX(-50%);
  color: #D7BD2C;
}

@media (max-width: 520px) {
  .weather {
    gap: 30px;
    flex-direction: column;
    align-items: center;
  }

  .block-wind {
    position: absolute;
    left: calc(50% + 80px);
  }
}
</style>
