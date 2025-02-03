<template>
  <v-row class="fill-height">
    <v-col>
      <!-- Calendario para mostrar los mealLogs -->
      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          v-model="today"
          :events="events"
          color="primary"
          type="month"
          @click:date="openForm"
        ></v-calendar>
      </v-sheet>

      <!-- Botón flotante para abrir el formulario (se usa la fecha actual: today[0]) -->
      <v-btn fab color="primary" class="ma-2" @click="openForm(today[0])">
        <v-icon>mdi-plus</v-icon>
      </v-btn>
    </v-col>
  </v-row>

  <!-- Diálogo con el formulario para registrar un nuevo mealLog -->
  <v-dialog v-model="showDialog" max-width="600">
    <v-card>
      <v-card-title>
        Nuevo Registro de Comida
      </v-card-title>
      <v-card-text>
        <v-form ref="mealForm">
          <v-text-field v-model="newMeal.email" label="Email"></v-text-field>
          <v-select
            v-model="newMeal.mealType"
            :items="mealTypes"
            label="Tipo de Comida"
          ></v-select>
          <!-- Campo de Fecha editable con control nativo -->
          <v-text-field
            v-model="newMeal.date"
            label="Fecha"
            type="date"
          ></v-text-field>
          <!-- Campo de Hora editable con control nativo -->
          <v-text-field
            v-model="newMeal.time"
            label="Hora"
            type="time"
          ></v-text-field>
          <v-textarea
            v-model="newMeal.foodItems"
            label="Comida"
            :rules="[ v => !!v || 'El campo Comida es obligatorio' ]"
          ></v-textarea>
          <v-text-field
            v-model="newMeal.calories"
            label="Calorías"
            type="number"
          ></v-text-field>
          <v-text-field
            v-model="newMeal.photoUrl"
            label="Foto URL"
          ></v-text-field>
          <v-textarea
            v-model="newMeal.notes"
            label="Notas"
          ></v-textarea>
          <v-text-field
            v-model="newMeal.location"
            label="Ubicación"
          ></v-text-field>
          <!-- Selector de estados de ánimo: se muestran en español, pero el valor se envía en inglés -->
          <v-select
            v-model="newMeal.mood"
            :items="moodOptions"
            label="Estado de Ánimo"
            item-title="text"
            item-value="value"
            :rules="[ v => !!v || 'El campo Estado de Ánimo es obligatorio' ]"
          ></v-select>
          <v-text-field
            v-model="newMeal.dietType"
            label="Tipo de Dieta"
          ></v-text-field>
          <v-text-field
            v-model="newMeal.diners"
            label="Cantidad de Comensales"
            type="number"
          ></v-text-field>
          <v-text-field
            v-model="newMeal.companions"
            label="Compañeros"
          ></v-text-field>
        </v-form>
      </v-card-text>
      <!-- Botones en horizontal -->
      <v-card-actions class="d-flex justify-end">
        <v-btn text class="mr-2" @click="showDialog = false">Cancelar</v-btn>
        <v-btn color="primary" @click="saveMealLog">Guardar</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      // VCalendar espera un array; "today" se inicializa con la fecha actual
      today: [new Date()],
      events: [],
      mealTypes: ['DESAYUNO', 'COMIDA', 'CENA', 'MERIENDA'],
      // Opciones para el selector de "Estado de Ánimo"
      moodOptions: [
        { value: 'OK', text: 'Bien' },
        { value: 'HAPPY', text: 'Feliz' },
        { value: 'NEUTRAL', text: 'Neutral' },
        { value: 'SAD', text: 'Triste' },
        { value: 'STRESSED', text: 'Estresado' },
        { value: 'TIRED', text: 'Cansado' },
        { value: 'ANXIOUS', text: 'Ansioso' },
        { value: 'UNWELL', text: 'Mal' },
        { value: 'FRUSTRATED', text: 'Frustrado' },
        { value: 'SICK', text: 'Enfermo' }
      ],
      // Modelo para el nuevo mealLog, con ubicación y compañeros por defecto
      newMeal: {
        email: 'joselitosbd@gmail.com',
        mealType: '',
        date: '',
        time: '', // Se asignará la hora actual al abrir el formulario
        foodItems: '',
        calories: null,
        photoUrl: '',
        notes: '',
        location: 'Casa Alzina',     // Valor por defecto
        mood: '',
        dietType: '',
        diners: 1,
        companions: 'Miriam, Jose A',  // Valor por defecto
        createdAt: new Date().toISOString()
      },
      showDialog: false
    }
  },
  mounted() {
    this.fetchMealLogs()
  },
  methods: {
    fetchMealLogs() {
      // Extraemos el año y el mes del primer elemento del array "today"
      const current = this.today[0]
      const year = current.getFullYear()
      const month = current.getMonth() + 1

      axios
        .get('http://localhost:8080/api/v1/meal-logs', { params: { year, month } })
        .then(response => {
          // Convertimos cada mealLog en un objeto "event" para el calendario
          this.events = response.data.map(meal => {
            const startDate = new Date(meal.date + 'T' + meal.time)
            const endDate = new Date(startDate.getTime() + 60 * 60 * 1000) // 1 hora después
            return {
              title: meal.foodItems,
              start: startDate,
              end: endDate,
              color: this.getMoodColor(meal.mood)
            }
          })
        })
        .catch(error => {
          console.error('Error fetching meal logs:', error)
        })
    },
    getMoodColor(mood) {
      const moodColors = {
        OK: 'green',
        HAPPY: 'blue',
        NEUTRAL: 'grey',
        SAD: 'purple',
        STRESSED: 'red',
        TIRED: 'orange',
        ANXIOUS: 'pink',
        UNWELL: 'brown',
        FRUSTRATED: 'yellow',
        SICK: 'black'
      }
      return moodColors[mood.toUpperCase()] || 'grey'
    },
    openForm(date) {
      // Al abrir el formulario, asignamos la fecha y la hora actuales
      const now = new Date()
      this.newMeal.date = now.toISOString().substring(0, 10)
      this.newMeal.time = now.toTimeString().substring(0, 8) // "HH:mm:ss"
      this.showDialog = true
    },
    saveMealLog() {
      if (!this.$refs.mealForm.validate()) {
        // Si la validación falla, no se continúa con la petición
        return;
      }
      axios
        .post('http://localhost:8080/api/v1/meal-logs', this.newMeal)
        .then(response => {
          this.showDialog = false
          // Actualizamos los eventos del mes actual
          this.fetchMealLogs()
          this.resetNewMeal()
        })
        .catch(error => {
          console.error('Error saving meal log:', error)
        })
    },
    resetNewMeal() {
      this.newMeal = {
        email: 'joselitosbd@gmail.com',
        mealType: '',
        date: '',
        time: '',
        foodItems: '',
        calories: null,
        photoUrl: '',
        notes: '',
        location: 'Casa Alzina',
        mood: '',
        dietType: '',
        diners: 2,
        companions: 'Miriam, Jose A',  // Valor por defecto
        createdAt: new Date().toISOString()
      }
    }
  }
}
</script>

<style scoped>
.ma-2 {
  position: fixed;
  right: 20px;
  bottom: 20px;
}
</style>