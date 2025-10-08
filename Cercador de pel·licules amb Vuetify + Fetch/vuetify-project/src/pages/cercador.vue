<template>
  <v-container fluid>

    <!-- Toolbar amb imatge i fons degradat -->
    <v-card height="220" class="pa-0" flat>
      <v-toolbar
        dark
        flat
        height="220"
        class="toolbar-bg d-flex flex-column justify-center px-6"
      >
        <v-btn icon class="mb-3" variant="text" color="white">
          <v-icon size="28">mdi-menu</v-icon>
        </v-btn>
        <v-toolbar-title class="text-h4 font-weight-bold white--text">
          Cercador de Pel·lícules
        </v-toolbar-title>
        <v-spacer />
        <v-btn icon variant="text" color="white">
          <v-icon size="28">mdi-export</v-icon>
        </v-btn>
      </v-toolbar>
    </v-card>

    <!-- Buscador -->
    <v-row
      class="mt-10 py-6 px-4 rounded-lg search-container"
      align="center"
      justify="center"
    >
      <v-col cols="12" md="6">
        <v-text-field
          v-model="textCerca"
          label="Escriu una pel·lícula..."
          prepend-inner-icon="mdi-magnify"
          clearable
          rounded
          outlined
          color="deep-orange accent-4"
          class="elevation-4"
          hide-details
          @keyup.enter="enviarPeticio"
          :disabled="loading"
        />
      </v-col>
      <v-col cols="auto" class="d-flex align-center">
        <v-btn
          color="deep-orange accent-4" 
          variant="elevated"
          @click="enviarPeticio"
          prepend-icon="mdi-movie-search"
          class="search-btn"
          :loading="loading"
          :disabled="loading"
        >
          Cercar
        </v-btn>
      </v-col>
    </v-row>

    <!-- Resultats -->
    <v-row class="mt-6" dense>
      <v-col
        cols="12"
        sm="6"
        md="4"
        lg="3"
        v-for="actual in pelicules"
        :key="actual.imdbID"
      >
        <v-card class="mx-auto" max-width="344" elevation="4" rounded>
          <v-img
            :src="actual.Poster !== 'N/A' ? actual.Poster : 'https://via.placeholder.com/300x450?text=No+Image'"
            height="400"
            cover
          />
          <v-card-title class="text-h6 font-weight-bold">{{ actual.Title }}</v-card-title>
          <v-card-subtitle class="grey--text">{{ actual.Year }}</v-card-subtitle>
          <v-card-actions>
            <v-btn
              color="deep-orange accent-4"
              variant="outlined"
              class="white--text"
              @click="demanarMesInfo(actual.imdbID); dialog = true"
            >
              Més informació
            </v-btn>
            <v-spacer />
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- Diàleg amb informació detallada -->
    <v-dialog v-model="dialog" max-width="600">
      <v-card>
        <v-card-title class="text-h6 font-weight-bold">
          {{ infoDetallada?.Title || 'Títol no disponible' }}
        </v-card-title>
        <v-card-subtitle class="grey--text">
          {{ infoDetallada?.Year }} • {{ infoDetallada?.Genre }}
        </v-card-subtitle>
        <v-card-text>
          <v-row>
            <v-col cols="12" md="4">
              <v-img
                :src="infoDetallada?.Poster !== 'N/A' ? infoDetallada?.Poster : 'https://via.placeholder.com/300x450?text=No+Image'"
                height="250"
              />
            </v-col>
            <v-col cols="12" md="8">
              <p><strong>Director:</strong> {{ infoDetallada?.Director }}</p>
              <p><strong>Actors:</strong> {{ infoDetallada?.Actors }}</p>
              <p><strong>Sinopsi:</strong> {{ infoDetallada?.Plot }}</p>
              <p><strong>Released:</strong> {{ infoDetallada.Released }}</p>
              <p><strong>Runtime:</strong> {{ infoDetallada.Runtime }}</p>
              <p style="display: flex; align-items: center; gap: 6px;">
                <strong>IMDB:</strong>
                <span>{{ infoDetallada?.imdbRating }}</span>
                <span class="rating-stars" style="display: flex; gap: 4px; margin-left: 8px;">
                  <v-icon
                    v-for="n in 5"
                    :key="n"
                    :color="n <= ratingToStars(infoDetallada.imdbRating) ? 'orange darken-3' : 'grey lighten-3'"
                    small
                    class="star-bordered"
                  >
                    {{ n <= ratingToStars(infoDetallada.imdbRating) ? 'mdi-star' : 'mdi-star-outline' }}
                  </v-icon>
                </span>
              </p>
              <p><strong>Escritors:</strong> {{ infoDetallada?.Writer }}</p>
              <p><strong>Country:</strong> {{ infoDetallada?.Country }}</p>
              <p style="display: flex; align-items: center; gap: 12px;">
                <strong>BoxOffice:</strong>
                <span class="boxoffice-amount">{{ infoDetallada?.BoxOffice }}</span>
                <div
                  v-if="infoDetallada?.BoxOffice && infoDetallada?.BoxOffice !== 'N/A'"
                  class="moving-stripe-bar"
                ></div>
              </p>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn text color="red" @click="dialog = false">Tancar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-container>
</template>

<script setup>
import { ref } from 'vue'

const dialog = ref(false)
const textCerca = ref('')
const pelicules = ref([])
const infoDetallada = ref({})
const loading = ref(false)

const API_KEY = 'fd954dba'
const BASE_URL = 'http://www.omdbapi.com/'

async function enviarPeticio() {
  if (!textCerca.value.trim()) {
    pelicules.value = []
    return
  }
  loading.value = true
  try {
    const response = await fetch(`${BASE_URL}?s=${encodeURIComponent(textCerca.value)}&apikey=${API_KEY}`)
    const data = await response.json()
    pelicules.value = data.Search || []
  } finally {
    loading.value = false
  }
}

async function demanarMesInfo(id) {
  const response = await fetch(`${BASE_URL}?i=${id}&apikey=${API_KEY}`)
  const data = await response.json()
  infoDetallada.value = data
}

function ratingToStars(rating) {
  if (!rating || rating === 'N/A') return 0;
  const num = parseFloat(rating);
  if (isNaN(num)) return 0;
  return Math.round(num / 2); // Convertim escala 0-10 a 0-5 estrelles
}
</script>

<style scoped>
.toolbar-bg {
  background: linear-gradient(135deg, #6D4C41, #A1887F); /* marró fosc a marró clar */
  padding-left: 24px;
  padding-right: 24px;
}

.v-toolbar-title {
  color: #EFEBE9; /* beix clar */
}

/* Ombra per millorar visibilitat icones toolbar */
.v-toolbar .v-btn.v-btn--icon .v-icon {
  text-shadow: 0 0 6px rgba(0, 0, 0, 0.7);
}

.search-container {
  background: #EFEBE9; /* beix suau */
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.search-btn {
  background-color: #FF7043 !important; /* taronja coral */
  color: white !important;
  transition: background-color 0.3s ease;
}

.search-btn:hover {
  background-color: #D84315 !important; /* taronja més fosc al hover */
  color: white !important;
}

/* Centrar vertical botó cerca respecte a text-field */
.v-row > .d-flex.align-center {
  height: 56px; /* alçada del text-field */
}

/* Botons outlined més contrastats */
.v-btn[variant="outlined"] {
  border-color: #BF360C !important; /* vermell fosc */
  color: #BF360C !important;
  font-weight: 600;
}

.v-btn[variant="outlined"]:hover {
  background-color: #FFCCBC !important; /* taronja clar */
  color: #BF360C !important;
}

.v-card {
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(109, 76, 65, 0.15);
}

.v-card-title {
  font-weight: 600;
  color: #3E2723; /*


.v-card-subtitle {
  font-size: 0.9rem;
  color: #004D40; /* verd fosc */
}

.boxoffice-amount {
  color: #2e7d32; /* verd fosc */
  font-weight: 700;
  margin-right: 12px;
}

.moving-stripe-bar {
  height: 8px;
  border-radius: 4px;
  background: repeating-linear-gradient(
    45deg,
    #2e7d32 0,
    #2e7d32 10px,
    #66bb6a 10px,
    #66bb6a 20px
  );
  background-size: 40px 40px;
  animation: move-stripes 2s linear infinite;
  margin-top: 0;
  margin-left: 4px;
}

.star-bordered {
  /* Contorn negre per fer l'efecte de borde */
  text-shadow:
    -1px -1px 0 #000,
     1px -1px 0 #000,
    -1px  1px 0 #000,
     1px  1px 0 #000;
}

@keyframes move-stripes {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: 40px 0;
  }
}
</style>
