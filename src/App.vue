<script setup>
import { ref, reactive, onMounted, computed, watch } from 'vue'

// Definiamo le props
defineProps({
  title: String,
})

// Stato reattivo
const users = ref([])
let filteredUsers = ref([])
const isLoading = ref(false)
const error = ref(null)
const searchFilter = ref('')
const favoriteCount = ref(0)

const formState = reactive({
  username: '',
  email: '',
})

// Recupero dei dati all'avvio
fetch('https://jsonplaceholder.typicode.com/users')
  .then((response) => {
    if (!response.ok) {
      error.value = 'Failed to fetch users'
    }
    return response.json()
  })
  .then((data) => {
    // Simuliamo un piccolo delay
    setTimeout(() => {
      users.value = data
      filteredUsers.value = data
      isLoading.value = false
    }, 1000)
  })

// Funzione per filtrare (chiamata dall'input)
function applyFilter() {
  console.log('Filtering with:', searchFilter.value)

  if (searchFilter.value == '') {
    filteredUsers.value = users.value
    return
  }

  filteredUsers.value = []
  for (let i = 0; i < users.value.length; i++) {
    if (users.value[i].name.includes(searchFilter.value)) {
      filteredUsers.value.push(users.value[i])
    }
  }
}

watch(searchFilter, (newVal) => {
  console.log('Filtro cambiato in:', newVal)
})

function toggleFavorite(user) {
  if (user.isFavorite) {
    user.isFavorite = false
    favoriteCount.value--
  } else {
    user.isFavorite = true
    favoriteCount.value++
  }
}

// Mostra un alert ogni volta che il conteggio cambia
watch(favoriteCount, (newCount) => {
  if (newCount > 0) {
    alert('Ora hai ' + newCount + ' utenti preferiti!')
  }
})
</script>

<template>
  <div class="user-dashboard">
    <h1>{{ title }}</h1>
    <div class="controls">
      <input
        v-model="searchFilter"
        @input="applyFilter"
        type="text"
        placeholder="Cerca utente per nome..."
      />
      <p>Preferiti selezionati: {{ favoriteCount }}</p>
    </div>

    <div v-if="isLoading" class="loading">Caricamento in corso...</div>

    <div v-if="error" class="error">{{ error }}</div>

    <ul v-if="!isLoading && !error">
      <li v-for="user in filteredUsers" :class="{ favorite: user.isFavorite }">
        <div class="user-info">
          <h3>{{ user.name }}</h3>
          <p>{{ user.email }}</p>
        </div>
        <button @click="toggleFavorite(user)">
          {{ user.isFavorite ? 'Rimuovi dai Preferiti' : 'Aggiungi ai Preferiti' }}
        </button>
      </li>
    </ul>

    <div v-if="filteredUsers.length === 0 && !isLoading" class="no-results">
      Nessun utente trovato con il filtro "{{ searchFilter }}".
    </div>
  </div>
</template>

<style scoped>
.user-dashboard {
  font-family: sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

input {
  padding: 8px;
  width: 60%;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  background: #f9f9f9;
  border: 1px solid #eee;
  margin-bottom: 10px;
  padding: 15px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

li.favorite {
  border-color: #ffd700;
  background-color: #fffef0;
}

.user-info h3 {
  margin: 0 0 5px 0;
}

.user-info p {
  margin: 0;
  color: #666;
}

button {
  padding: 8px 12px;
  cursor: pointer;
}

.loading,
.error,
.no-results {
  text-align: center;
  margin-top: 30px;
}
</style>
