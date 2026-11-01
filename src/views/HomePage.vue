<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Prakiraan Cuaca Jakarta</ion-title>
      </ion-toolbar>
    </ion-header>


  <ion-content class= "weather-page">
    <div class="weather-container">

      <!-- Judul aplikasi-->
    <div class="page-heading">
      <h2>Informasi Cuaca Jakarta</h2>
      <p>Prakiraan temperatur per jam berdasarkan data Open-Meteo</p>
    </div>

    <!-- Informasi Lokasi-->
     <div class ="section-card">
      <h3>Informasi Lokasi</h3>

      <div class="location-grid">
        <div class="info-item">
          <span>Latitude</span>
          <strong>{{ location.latitude ?? '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Longitude</span>
          <strong>{{ location.longitude ?? '-' }}</strong>
        </div>

        <div class="info-item">
          <span>Ketinggian</span>
          <strong>{{ location.elevation ?? '-' }} meter</strong>
        </div>

        <div class="info-item">
          <span>Zona Waktu</span>
          <strong>{{ location.timezone || '-' }}</strong>
        </div>
      </div>
     </div>


    <!-- Tombol mengambil data -->
    <ion-button
      expanf="block"
      @click="ambilData"
      :disabled="loading"
      >
      {{ loading ? 'Memuat Data...' : 'Ambil Data Cuaca' }}
    </ion-button>

    <p v-if="loading" class="status">
      Sedang mengambil data dari Open-Meteo...
    </p>

    <p v-if="errorMessage" class="error">
      {{ errrorMessage }}
    </p>

<!-- Ringkasan temperatur -->
        <div
          v-if="weatherData.length > 0"
          class="section-card"
        >
          <h3>Ringkasan Temperatur</h3>

          <div class="summary-grid">
            <div class="summary-item">
              <span>Temperatur Tertinggi</span>
              <strong>{{ maxTemperature }} °C</strong>
            </div>

            <div class="summary-item">
              <span>Temperatur Terendah</span>
              <strong>{{ minTemperature }} °C</strong>
            </div>

            <div class="summary-item">
              <span>Temperatur Rata-rata</span>
              <strong>{{ averageTemperature }} °C</strong>
            </div>
          </div>
        </div>

        <!-- Tabel data cuaca -->
        <div
          v-if="weatherData.length > 0"
          class="section-card table-card"
        >
          <h3>Data Temperatur Per Jam</h3>

          <p class="table-description">
            Menampilkan 10 data pengukuran pertama.
          </p>

          <div class="table-container">
            <table>
              <thead>
                <tr>
                  <th>No.</th>
                  <th>Tanggal</th>
                  <th>Waktu (GMT)</th>
                  <th>Temperatur</th>
                  <th>Kategori</th>
                </tr>
              </thead>

              <tbody>
                <tr
                  v-for="(w, index) in weatherData"
                  :key="index"
                >
                  <td>{{ index + 1 }}</td>
                  <td>{{ formatTanggal(w.time) }}</td>
                  <td>{{ formatWaktu(w.time) }}</td>
                  <td>{{ w.temperature_2m ?? '-' }} °C</td>
                  <td>
                    <span
                      class="category"
                      :class="kategoriClass(w.temperature_2m)"
                    >
                      {{ kategoriSuhu(w.temperature_2m) }}
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Sumber data -->
        <div class="footer-info">
          <p>
            Sumber data:
            <a
              href="https://open-meteo.com/"
              target="_blank"
              rel="noopener noreferrer"
            >
              Open-Meteo API
            </a>
          </p>
        </div>

      </div>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import axios from 'axios';

interface WeatherData {
  time: string;
  temperature_2m: number | null;
}

export default defineComponent({
  name: 'HomePage',

  data() {
    return {
      weatherData: [] as WeatherData[],

      location: {
        latitude: null as number | null,
        longitude: null as number | null,
        elevation: null as number | null,
        timezone: ''
      },

      loading: false,
      errorMessage: ''
    };
  },

  computed: {
    maxTemperature(): string {
      const values = this.weatherData
        .map(w => w.temperature_2m)
        .filter((v): v is number => v !== null);

      if (values.length === 0) return '-';

      return Math.max(...values).toFixed(1);
    },

    minTemperature(): string {
      const values = this.weatherData
        .map(w => w.temperature_2m)
        .filter((v): v is number => v !== null);

      if (values.length === 0) return '-';

      return Math.min(...values).toFixed(1);
    },

    averageTemperature(): string {
      const values = this.weatherData
        .map(w => w.temperature_2m)
        .filter((v): v is number => v !== null);

      if (values.length === 0) return '-';

      const total = values.reduce(
        (sum, value) => sum + value,
        0
      );

      return (total / values.length).toFixed(1);
    }
  },

  methods: {
    async ambilData() {
      this.loading = true;
      this.errorMessage = '';

      try {
        const url =
          'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

        const response = await axios.get(url);

        const data = response.data;

        // Mengambil informasi lokasi
        this.location = {
          latitude: data.latitude,
          longitude: data.longitude,
          elevation: data.elevation,
          timezone: data.timezone
        };

        // Mengambil data waktu dan temperatur
        const waktu = data.hourly.time;
        const temperatur = data.hourly.temperature_2m;

        // Menggabungkan waktu dan temperatur
        this.weatherData = waktu
          .slice(0, 10)
          .map((time: string, index: number) => ({
            time: time,
            temperature_2m: temperatur[index]
          }));

      } catch (error) {
        this.errorMessage =
          'Gagal mengambil data cuaca. Periksa koneksi internet Anda.';

        console.error(error);

      } finally {
        this.loading = false;
      }
    },

    formatTanggal(time: string): string {
      return time.split('T')[0];
    },

    formatWaktu(time: string): string {
      return time.split('T')[1];
    },

    kategoriSuhu(temperature: number | null): string {
      if (temperature === null) return 'Tidak tersedia';

      if (temperature < 28) return 'Sejuk';

      if (temperature < 31) return 'Hangat';

      return 'Panas';
    },

    kategoriClass(temperature: number | null): string {
      if (temperature === null) return 'category-neutral';

      if (temperature < 28) return 'category-cool';

      if (temperature < 31) return 'category-warm';

      return 'category-hot';
    }
  },

  mounted() {
    this.ambilData();
  }
});
</script>

<style scoped>
.weather-page {
  --background: #f3f6fb;
  background: #f3f6fb;
}

.weather-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 18px;
  color: #263445;
}

.page-heading {
  text-align: center;
  margin: 10px 0 25px;
}

.page-heading h2 {
  font-size: 23px;
  font-weight: 700;
  color: #1e293b;
}

.page-heading p {
  color: #64748b;
  line-height: 1.6;
}

.section-card {
  background: #ffffff;
  border-radius: 13px;
  padding: 18px;
  margin-bottom: 20px;
  box-shadow: 0 3px 12px rgba(0, 0, 0, 0.05);
}

.section-card h3 {
  color: #1e293b;
  font-size: 17px;
  font-weight: 700;
  margin-top: 0;
  margin-bottom: 18px;
}

.location-grid,
.summary-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.info-item,
.summary-item {
  background: #f5f8fc;
  padding: 13px;
  border-radius: 9px;
}

.info-item span,
.summary-item span {
  display: block;
  color: #64748b;
  font-size: 12px;
  margin-bottom: 7px;
}

.info-item strong,
.summary-item strong {
  color: #1e293b;
  font-size: 15px;
}

.summary-item strong {
  color: #2563eb;
  font-size: 20px;
}

ion-button {
  --background: #2563eb;
  --background-hover: #1d4ed8;
  --border-radius: 9px;
  font-weight: 600;
  margin-bottom: 20px;
}

.status {
  text-align: center;
  color: #2563eb;
  font-weight: 600;
}

.error {
  text-align: center;
  color: #dc2626;
  font-weight: 600;
}

.table-description {
  color: #64748b;
  font-size: 13px;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  min-width: 550px;
  border-collapse: collapse;
  text-align: center;
  background: #ffffff;
}

th {
  background: #2563eb;
  color: #ffffff;
  padding: 13px 10px;
  white-space: nowrap;
  font-size: 13px;
}

td {
  padding: 12px 10px;
  color: #334155;
  border-bottom: 1px solid #e2e8f0;
  font-size: 13px;
  white-space: nowrap;
}

tbody tr:nth-child(even) {
  background: #f8fafc;
}

tbody tr:hover {
  background: #eff6ff;
}

.category {
  display: inline-block;
  padding: 5px 9px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
}

.category-cool {
  background: #dbeafe;
  color: #1d4ed8;
}

.category-warm {
  background: #fef3c7;
  color: #92400e;
}

.category-hot {
  background: #fee2e2;
  color: #b91c1c;
}

.category-neutral {
  background: #e2e8f0;
  color: #475569;
}

.footer-info {
  text-align: center;
  color: #64748b;
  font-size: 12px;
  padding: 10px 0 20px;
}

.footer-info a {
  color: #2563eb;
  text-decoration: none;
}

@media (min-width: 700px) {
  .location-grid,
  .summary-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
</style>


