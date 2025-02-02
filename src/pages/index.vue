<template>
  <v-container>
    <!-- Calendario -->
    <v-sheet height="600">
      <v-calendar
        v-model="currentDate"
        type="month"
        :events="events"
        @click:event="openEvent"
        @click:date="openForm"
      />
    </v-sheet>

    <!-- Botón flotante "+" -->
    <v-btn 
      color="primary" 
      fab 
      fixed 
      bottom 
      right 
      @click="openForm(currentDate)"
    >
      <v-icon>mdi-plus</v-icon>
    </v-btn>

    <!-- Diálogo para crear un nuevo registro -->
    <v-dialog v-model="dialog" max-width="500">
      <v-card>
        <v-card-title>Nuevo Registro</v-card-title>
        <v-card-text>
          <v-text-field v-model="newMeal.date" label="Fecha" readonly></v-text-field>
          <v-select v-model="newMeal.mealType" :items="mealTypes" label="Tipo de comida"></v-select>
          <v-select v-model="newMeal.mood" :items="moods" label="Estado de ánimo"></v-select>
          <v-textarea v-model="newMeal.foodItems" label="Comida"></v-textarea>
          <v-text-field v-model="newMeal.location" label="Ubicación"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="saveMeal">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      currentDate: new Date().toISOString().split('T')[0], // Fecha actual en formato YYYY-MM-DD
      dialog: false,
      events: [],
      newMeal: {
        email: "joselitosbd@gmail.com",
        mealType: "",
        date: "",
        time: "20:00:00",
        foodItems: "",
        calories: null,
        photoUrl: "",
        notes: "",
        location: "",
        mood: "",
        dietType: null,
        diners: 1,
        companions: "",
        createdAt: new Date().toISOString()
      },
      mealTypes: ["DESAYUNO", "ALMUERZO", "CENA", "SNACK"],
      moods: ["OK", "HAPPY", "NEUTRAL", "SAD", "STRESSED", "TIRED", "ANXIOUS", "UNWELL", "FRUSTRATED", "SICK"],
    };
  },
  mounted() {
    this.fetchMeals();
  },
  methods: {
    async fetchMeals() {
      const year = this.currentDate.substring(0, 4); // Extrae el año
      const month = this.currentDate.substring(5, 7); // Extrae el mes
      try {
        const response = await axios.get(`http://localhost:8080/meal-logs`, {
          params: { year, month }
        });
        this.events = response.data.map(meal => ({
          name: meal.foodItems,
          start: meal.date,
          color: this.getMoodColor(meal.mood)
        }));
      } catch (error) {
        console.error("Error obteniendo registros:", error);
      }
    },
    openEvent(event) {
      alert(`Detalles: ${event.name}`);
    },
    openForm(date) {
      this.newMeal.date = date;
      this.dialog = true;
    },
    async saveMeal() {
      try {
        await axios.post("http://localhost:8080/meal-logs", this.newMeal);
        this.dialog = false;
        this.fetchMeals(); // Recargar eventos
      } catch (error) {
        console.error("Error al guardar:", error);
      }
    },
    getMoodColor(mood) {
      const moodColors = {
        OK: "green",
        HAPPY: "blue",
        NEUTRAL: "grey",
        SAD: "purple",
        STRESSED: "red",
        TIRED: "orange",
        ANXIOUS: "pink",
        UNWELL: "brown",
        FRUSTRATED: "yellow",
        SICK: "black"
      };
      return moodColors[mood] || "grey";
    }
  }
};
</script>

<style scoped>
.v-btn {
  position: fixed;
  right: 20px;
  bottom: 20px;
}
</style>
