<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Aplicación del Clima</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <!-- Campo de entrada para la ubicación -->
      <ion-item>
        <ion-label position="floating">Ingresa una ubicación</ion-label> <br>
        <ion-input v-model="location" placeholder="Ej: San José"></ion-input>
      </ion-item>

      <!-- Botones para obtener el clima -->
      <ion-button expand="full" @click="getWeather">Obtener Clima</ion-button>
      <ion-button expand="full" @click="getCurrentLocationWeather">Usar Ubicación Actual</ion-button>

      <!-- Tarjeta con los datos del clima actual -->
      <ion-card v-if="weatherData">
        <ion-card-header>
          {{ weatherData.resolvedAddress }}
        </ion-card-header>
        <ion-card-content>
          <p>Temperatura: {{ weatherData.currentConditions.temp }}°C</p>
          <p>Velocidad del Viento: {{ weatherData.currentConditions.windspeed }} km/h</p>
          <p>Precipitación: {{ weatherData.currentConditions.precip }}%</p>
          <p>Condiciones: {{ weatherData.currentConditions.conditions }}</p>
        </ion-card-content>
      </ion-card>

      <!-- Lista de las últimas 24 horas -->
      <ion-list v-if="weatherData">
        <ion-list-header>
          Últimas 24 Horas
        </ion-list-header>
        <ion-item v-for="hour in weatherData.hours" :key="hour.datetime">
          <ion-label>
            {{ hour.datetime }}: {{ hour.temp }}°C, {{ hour.conditions }}
          </ion-label>
        </ion-item>
      </ion-list>

      <!-- Lista de las próximas 24 horas -->
      <ion-list v-if="weatherData">
        <ion-list-header>
          Próximas 24 Horas
        </ion-list-header>
        <ion-item v-for="hour in weatherData.nextHours" :key="hour.datetime">
          <ion-label>
            {{ hour.datetime }}: {{ hour.temp }}°C, {{ hour.conditions }}
          </ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script>
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonButton,
  IonCard,
  IonCardHeader,
  IonCardContent,
  IonList,
  IonListHeader,
} from "@ionic/vue";
import axios from "axios";
import { Geolocation } from "@capacitor/geolocation";

export default {
  components: {
    IonPage,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonItem,
    IonLabel,
    IonInput,
    IonButton,
    IonCard,
    IonCardHeader,
    IonCardContent,
    IonList,
    IonListHeader,
  },
  data() {
    return {
      location: "",
      weatherData: null,
    };
  },
  methods: {
    async getWeather() {
      const apiKey = "68A7SHNHVB63URCH4CELS3NMU";
      const location = encodeURIComponent(this.location); // Codifica la ubicación
      const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${location}?unitGroup=metric&key=${apiKey}&contentType=json`;

      try {
        const response = await axios.get(url);
        this.weatherData = response.data;
        this.weatherData.nextHours = this.weatherData.days[0].hours.slice(0, 24);
      } catch (error) {
        console.error("Error al obtener los datos del clima:", error);
        alert("Error al obtener los datos del clima. Por favor, intenta de nuevo.");
      }
    },
    async getCurrentLocationWeather() {
      try {
        const coordinates = await Geolocation.getCurrentPosition();
        const apiKey = "68A7SHNHVB63URCH4CELS3NMU";
        const url = `https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/${coordinates.coords.latitude},${coordinates.coords.longitude}?unitGroup=metric&key=${apiKey}&contentType=json`;

        const response = await axios.get(url);
        this.weatherData = response.data;
        this.weatherData.nextHours = this.weatherData.days[0].hours.slice(0, 24);
      } catch (error) {
        console.error("Error al obtener el clima de la ubicación actual:", error);
        alert("Error al obtener el clima de la ubicación actual. Por favor, intenta de nuevo.");
      }
    },
  },
};
</script>