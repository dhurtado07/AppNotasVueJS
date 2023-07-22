<template>
  <main class="flex justify-center align-middle">
    <div class="w-full sm:w-3/5 md:w-3/5 lg:w-2/5 bg-slate-200 my-3 drop-shadow-md rounded-md">
      <div class="p-2">
        <input v-model="note" placeholder="Escriba una nota"
          class="w-full bg-zinc-100 py-2 px-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md">

        <input type="date" v-model="date"
          class="w-full bg-zinc-100 py-2 px-2 mt-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md">

        <label for="userSelect">Asignar a un usuario:</label>
        <br>
        <select v-model="selectedUserId" id="userSelect"
          class="w-full bg-zinc-100 py-2 px-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md">
          <option value="">Seleccione un usuario</option>
          <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
        </select>
        <button v-if="isEditing === false"
          class="w-36 bg-slate-400 rounded-md m-1 py-2 px-1 text-white hover:bg-zinc-500 hover:drop-shadow-md"
          @click="createTask">Add</button>
        <button v-else
          class="w-36 bg-slate-400 rounded-md m-1 py-2 px-1 text-white hover:bg-zinc-500 hover:drop-shadow-md"
          @click="updateTask">Update</button>
      </div>
      <div class="p-2">
        <div class="flex">
          <div class="w-1/3">
            <input v-model="searchQuery" placeholder="Buscar nota"
              class="w-full bg-zinc-100 py-2 px-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md">
          </div>
          <div class="w-1/3 ml-2">
            <input type="date" v-model="dateFilter"
              class="w-full bg-zinc-100 py-2 px-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md"
              :disabled="!tasks.length">
          </div>
          <div class="w-1/3 ml-2">
            <select v-model="userFilter"
              class="w-full bg-zinc-100 py-2 px-2 rounded outline-1 outline outline-zinc-300 focus:outline-none focus:drop-shadow-md"
              :disabled="!tasks.length">
              <option value="">Todos los usuarios</option>
              <option v-for="user in users" :key="user.id" :value="user.id">{{ user.name }}</option>
            </select>
          </div>
        </div>
        <ul>
          <li v-for="ts in filteredTasks" :key="ts.id"
            class="bg-slate-300 py-2 px-3 my-1 rounded-md">
            <div class="flex justify-between items-center">
              <div>
                <h3 class="font-bold">{{ ts.id }}</h3>
                <p class="mb-1">{{ ts.note }}</p>
                <p class="mb-2">{{ ts.date }}</p> <!-- Agregamos un espacio adicional entre la nota y la fecha -->
                <p v-if="ts.userId">Assigned to: {{ getUserName(ts.userId) }}</p>
              </div>
              <div>
                <button class="bg-blue-500 text-white rounded-md py-2 px-1" @click="editTask(ts.id)">Edit</button>
                <button class="bg-red-500 text-white rounded-md py-2 px-1 ml-1"
                  @click="deleteTask(ts.id)">Delete</button>
              </div>
            </div>
          </li>
        </ul>
        <p v-if="filteredTasks.length === 0 && tasks.length > 0" class="text-center">No se encontraron tareas que coincidan con los criterios de búsqueda.</p>
        <p v-if="tasks.length === 0" class="text-center">No hay notas creadas aún.</p>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

// Entidad Tarea
let id = 1
let isEditing = false
let idEditTask = null
const tasks = ref([])

const date = ref('')
const note = ref('')
const selectedUserId = ref('')
const searchQuery = ref('')
const dateFilter = ref('')
const userFilter = ref('')

async function createTask() {
  const newTask = {
    "id": id++,
    "note": note.value,
    "date": date.value,
    "userId": selectedUserId.value
  };

  try {
    const response = await fetch('http://localhost:3001/tasks', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(newTask)
    });

    if (response.ok) {
      tasks.value.push(newTask);
      saveData();
      note.value = '';
      date.value = '';
      selectedUserId.value = '';
    } else {
      console.error("Error al crear la tarea. Respuesta del servidor:", response.status);
    }
  } catch (error) {
    console.error("Error al crear la tarea:", error);
  }
}
function editTask(id) {
  isEditing = true;
  idEditTask = id;
  const currentTask = tasks.value.find(tsk => tsk.id === id);
  note.value = currentTask.note;
  date.value = currentTask.date;
  selectedUserId.value = currentTask.userId;
}
async function updateTask() {
  const editedTask = {
    "id": idEditTask,
    "note": note.value,
    "date": date.value,
    "userId": selectedUserId.value
  };

  try {
    const response = await fetch(`http://localhost:3001/tasks/${idEditTask}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(editedTask)
    });

    if (response.ok) {
      const index = tasks.value.findIndex(tsk => tsk.id === idEditTask);
      if (index !== -1) {
        tasks.value[index] = editedTask;
        saveData();
        note.value = '';
        date.value = '';
        selectedUserId.value = '';
        isEditing = false;
      }
    } else {
      console.error("Error al actualizar la tarea. Respuesta del servidor:", response.status);
    }
  } catch (error) {
    console.error("Error al actualizar la tarea:", error);
  }
}

async function deleteTask(id) {
  try {
    const response = await fetch(`http://localhost:3001/tasks/${id}`, {
      method: 'DELETE'
    });

    if (response.ok) {
      tasks.value = tasks.value.filter(tsk => tsk.id !== id);
      saveData();
    } else {
      console.error("Error al eliminar la tarea. Respuesta del servidor:", response.status);
    }
  } catch (error) {
    console.error("Error al eliminar la tarea:", error);
  }
}

// Entidad Usuario
const users = ref([])

function getUserName(userId) {
  const user = users.value.find(user => user.id === userId)
  return user ? user.name : "Unknown"
}

// Computed para filtrar tareas
const filteredTasks = computed(() => {
  let filtered = tasks.value

  if (searchQuery.value) {
    const searchQueryLC = searchQuery.value.toLowerCase()
    filtered = filtered.filter(ts => ts.note.toLowerCase().includes(searchQueryLC))
  }

  if (dateFilter.value) {
    filtered = filtered.filter(ts => ts.date === dateFilter.value)
  }

  if (userFilter.value !== '') {
    filtered = filtered.filter(ts => ts.userId === parseInt(userFilter.value))
  }

  return filtered
})

// Función para cargar los datos desde db.json
async function fetchData() {
  try {
    const response = await fetch('http://localhost:3001/tasks')
    const data = await response.json()
    tasks.value = data
  } catch (error) {
    console.error("Error al obtener las tareas:", error)
  }

  try {
    const userResponse = await fetch('http://localhost:3001/users')
    const userData = await userResponse.json()
    users.value = userData
  } catch (error) {
    console.error("Error al obtener los usuarios:", error)
  }
}

// Función para guardar los datos en db.json
async function saveData() {
  try {
    await fetch('http://localhost:3001/tasks', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(tasks.value)
    })
  } catch (error) {
    console.error("Error al guardar las tareas:", error)
  }
}

// Ejecutamos fetchData al cargar el componente para obtener los datos iniciales
onMounted(fetchData)
</script>
