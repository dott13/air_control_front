<template>
  <div class="container">
    <h1>🌬️ Air Quality Dashboard</h1>

    <div v-if="error" class="error">{{ error }}</div>

    <div class="stats-container">
      <div class="stat-card">
        <h2>💧 Humidity</h2>
        <p class="value">{{ reading.humidity.toFixed(1) }} %</p>
      </div>
      <div class="stat-card">
        <h2>🌡️ Temperature</h2>
        <p class="value">{{ reading.temperature.toFixed(1) }} °C</p>
      </div>
      <div class="stat-card">
        <h2>🛰️ CO₂</h2>
        <p class="value">{{ reading.co2 }} ppm</p>
      </div>
    </div>

    <p class="updated">Last updated: {{ formattedTime }}</p>

    <div class="buttons">
      <button class="test" @click="testConnection">Test Connection</button>
      <button class="open" @click="openWindow">Open Window</button>
    </div>

    <p class="status">{{ status }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";

// ————————— CONFIG —————————
const API_URL = "http://localhost:3000/api/readings/latest";
const TEST_URL = "http://localhost:3000/api/test";
const OPEN_URL = "http://localhost:3000/api/window/open";

// ————————— STATE ——————————
const reading = ref({
  humidity: 0,
  temperature: 0,
  co2: 0,
  createdAt: new Date().toISOString(),
});
const status = ref("");
const error = ref("");

// ———————— MOCK DATA ————————
function setMockData() {
  reading.value = {
    humidity: 45.2,
    temperature: 22.5,
    co2: 415,
    createdAt: new Date().toISOString(),
  };
}

// ———————— FETCHING ————————
async function fetchReading() {
  try {
    const res = await axios.get(API_URL);
    reading.value = res.data;
    error.value = "";
  } catch (e) {
    error.value = "⚠️ Failed to fetch data, showing mock values";
    setMockData();
  }
}

// —————— TEST & OPEN ——————
async function testConnection() {
  status.value = "Testing…";
  try {
    await axios.get(TEST_URL);
    status.value = "✅ Server reachable!";
  } catch {
    status.value = "❌ Server test failed";
  }
}

async function openWindow() {
  status.value = "Opening window…";
  try {
    await axios.post(OPEN_URL);
    status.value = "✅ Open command sent!";
  } catch {
    status.value = "❌ Failed to send open command";
  }
}

// ————— LIFECYCLE —————
onMounted(() => {
  setMockData(); // show mock immediately
  fetchReading(); // then try real data
  setInterval(fetchReading, 60_000); // every 60 s
});

// —————— HELPERS ——————
const formattedTime = computed(() => {
  return new Date(reading.value.createdAt).toLocaleString();
});
</script>

<style>
.container {
  max-width: 480px;
  margin: 2rem auto;
  padding: 0 1rem;
  font-family: "Segoe UI", sans-serif;
  text-align: center;
}
h1 {
  margin-bottom: 1rem;
}
.stats-container {
  display: flex;
  justify-content: space-between;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}
.stat-card {
  flex: 1;
  background: #f7f9fc;
  border: 1px solid #dde3ea;
  border-radius: 8px;
  padding: 1rem;
}
.stat-card h2 {
  margin: 0 0 0.5rem;
  font-size: 1rem;
}
.value {
  font-size: 1.5rem;
  font-weight: bold;
  margin: 0;
}
.updated {
  font-size: 0.8rem;
  color: #666;
  margin-bottom: 1rem;
}
.buttons {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}
button {
  padding: 0.6rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
}
button.test {
  background-color: #007bff;
  color: white;
}
button.open {
  background-color: #28a745;
  color: white;
}
button:hover {
  opacity: 0.9;
}
.status {
  font-size: 0.9rem;
  font-weight: 500;
  color: #333;
}
.error {
  color: #c00;
  margin-bottom: 0.5rem;
}
</style>
