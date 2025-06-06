<template>
  <main>
    <div>
      <hr />
      <button @click="usersList">Récupérer les utilisateurs</button>
      <input type="text" v-model="searchTerm" placeholder="Rechercher par nom" @input="applyFiltersAndDisplay" />
    </div>

    <table>
      <thead>
      <tr>
        <th>Photo</th>
        <th>Nom</th>
        <th>Email</th>
        <th>Tel</th>
        <th @click="sortUsersByAge">
          Age <span>{{ ageArrow }}</span>
        </th>
        <th @click="toggleGenderFilter">
          Genre <span>{{ genderArrow }}</span>
        </th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="user in filteredUsers" :key="user.login.uuid">
        <td><img :src="user.picture.thumbnail" :alt="`Photo de ${user.name.first}`" /></td>
        <td>{{ user.name.first }} {{ user.name.last }}</td>
        <td>{{ user.email }}</td>
        <td>{{ user.phone }}</td>
        <td>{{ user.dob.age }}</td>
        <td>{{ user.gender === 'male' ? '👨' : '👩' }}</td>
      </tr>
      </tbody>
    </table>
  </main>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const users = ref([])
const searchTerm = ref('')
const ageSortAsc = ref(null)
const genderFilterState = ref(0)

const usersList = async () => {
  try {
    const response = await fetch('https://randomuser.me/api/?results=40')
    const data = await response.json()
    users.value.push(...data.results)
    applyFiltersAndDisplay()
  } catch (error) {
    console.error(error)
  }
}

const filteredUsers = ref([])

const normalize = str =>
    str.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g, '')

const applyFiltersAndDisplay = () => {
  let temp = [...users.value]

  if (genderFilterState.value === 1) {
    temp = temp.filter(u => u.gender === 'male')
  } else if (genderFilterState.value === 2) {
    temp = temp.filter(u => u.gender === 'female')
  }

  const term = normalize(searchTerm.value)
  if (term) {
    temp = temp.filter(u => {
      const fullName = normalize(`${u.name.first} ${u.name.last}`)
      return fullName.includes(term)
    })
  }

  if (ageSortAsc.value !== null) {
    temp.sort((a, b) =>
        ageSortAsc.value ? a.dob.age - b.dob.age : b.dob.age - a.dob.age
    )
  }

  filteredUsers.value = temp
}

const sortUsersByAge = () => {
  ageSortAsc.value =
      ageSortAsc.value === null ? true : ageSortAsc.value === true ? false : null
  applyFiltersAndDisplay()
}

const toggleGenderFilter = () => {
  genderFilterState.value = (genderFilterState.value + 1) % 3
  applyFiltersAndDisplay()
}

const ageArrow = computed(() => {
  return ageSortAsc.value === null ? '↕' : ageSortAsc.value ? '↑' : '↓'
})

const genderArrow = computed(() => {
  return genderFilterState.value === 0
      ? '↕'
      : genderFilterState.value === 1
          ? '♂'
          : '♀'
})
</script>

<style>
img {
  border-radius: 50%;
}
</style>
