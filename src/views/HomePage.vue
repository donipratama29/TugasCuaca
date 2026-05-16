<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="dark">
        <ion-title>Prakiraan Cuaca di Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content color="dark">
      <!-- Loading -->
      <div v-if="loading" class="loading">
        <ion-spinner name="dots"></ion-spinner>
        <p>Memuat data cuaca...</p>
      </div>

      <!-- Data Cuaca -->
      <ion-list v-else lines="none">
        <ion-card
          v-for="(w, index) in weatherData"
          :key="index"
          class="weather-card"
        >
          <ion-card-content>
            <h1 :style="{ color: getTempColor(w.temperature_2m) }">
              {{ formatTemp(w.temperature_2m) }}
            </h1>
            <p>{{ formatDate(w.time) }}</p>
          </ion-card-content>
        </ion-card>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

// Interface data cuaca
interface WeatherItem {
  time: string
  temperature_2m: number
}

// State
const weatherData = ref<WeatherItem[]>([])
const loading = ref(true)

// Ambil data saat halaman dibuka
onMounted(async () => {
  try {
    // API Jakarta sesuai soal
    const response = await axios.get(
      'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m'
    )

    // Ambil 10 data pertama
    const times = response.data.hourly.time
    const temps = response.data.hourly.temperature_2m

    // Gabungkan data
    weatherData.value = times.map((t: string, i: number) => ({
      time: t,
      temperature_2m: temps[i]
    }))
  } catch (err) {
    console.error('Error fetching data:', err)
  } finally {
    loading.value = false
  }
})

// Format tanggal Indonesia
const formatDate = (dateString: string) => {
  const date = new Date(dateString)

  return (
    date.toLocaleDateString('id-ID', {
      weekday: 'long',
      day: 'numeric',
      month: 'long',
      year: 'numeric'
    }) +
    ' pukul ' +
    date
      .toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit'
      })
      .replace(':', '.')
  )
}

// Format suhu dengan emoji
const formatTemp = (temp: number) => {
  let emoji = ''

  if (temp >= 38) emoji = '🔥'
  else if (temp >= 35) emoji = '🌞' 
  else if (temp >= 32) emoji = '🌤️'
  else if (temp >= 29) emoji = '⛅'
  else if (temp >= 26) emoji = '🌥️'
  else if (temp >= 23) emoji = '🌦️'
  else if (temp >= 20) emoji = '🌧️'
  else emoji = '⛈️'

  return `${emoji} ${temp.toFixed(1)}°C`
}

// Warna suhu berdasarkan temperatur
const getTempColor = (temp: number) => {
  if (temp >= 38) return '#ff3b30'      // Merah panas ekstrem
  else if (temp >= 35) return '#ff6b00' // Oranye panas
  else if (temp >= 32) return '#ff9500' // Jingga
  else if (temp >= 29) return '#ffd60a' // Kuning
  else if (temp >= 26) return '#34c759' // Hijau
  else if (temp >= 23) return '#0a84ff' // Biru muda
  else if (temp >= 20) return '#5e5ce6' // Biru
  else return '#8e8e93'                 // Abu dingin
}
</script>

<style scoped>
.loading {
  text-align: center;
  margin-top: 30px;
  color: white;
}

.weather-card {
  margin: 12px;
  border-radius: 16px;
  background: linear-gradient(135deg, #1e1e1e, #2d2d2d);
  color: white;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.4);
  transition: transform 0.2s ease;
}

.weather-card:hover {
  transform: scale(1.02);
}

h1 {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 10px;
}

p {
  font-size: 16px;
  color: #d0d0d0;
}
</style>