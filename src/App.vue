<script setup>
import { ref, onMounted, reactive } from 'vue';

const todos = ref([]);
const showUpdateModal = ref(false);
const editingItemId = ref(null);
const updateTitle = ref('');

const todo = reactive({
  title: '',
  completed: false,
});

function fetchTodos() {
  fetch(`https://jsonplaceholder.typicode.com/todos`)
    .then(res => res.json())
    .then(json => {
      todos.value = json;
    });
}

function onShowUpdateModal(item) {
  showUpdateModal.value = true;
  editingItemId.value = item.id;
  updateTitle.value = item.title;
}

function onSubmitUpdateModal() {
  fetch(`https://jsonplaceholder.typicode.com/todos/${editingItemId.value}`, {
    method: 'PUT',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      title: updateTitle.value,
      completed: todos.value.find(todo => todo.id === editingItemId.value).completed,
    }),
  })
    .then(res => res.json())
    .then((item) => {
      if (!item.id) {
        return;
      }
      showUpdateModal.value = false;
      fetchTodos();
      resetTodoObj();
    });
}

function resetTodoObj() {
  updateTitle.value = '';
}

onMounted(fetchTodos);
</script>

<template>
  <div class="container">
    <h1 class="text-2xl text-center font-bold">Bu sizning rejalaringiz</h1>
    <div v-for="todo of todos" :key="todo.id" class="xs:w-full md:w-1/2 xl:w-1/3 2xl:w-1/4 p-3">
      <div :class="{'bg-green-200': todo.completed, 'bg-red-200': !todo.completed}" class="rounded-lg cursor-pointer hover:shadow-md p-2 transition-shadow">
        <h5 class="my-2 text-lg hover:text-blue-500">{{ todo.title }}</h5>
        <p class="truncate text-gray-500 ttext-end">
          {{ todo.completed ? 'Done' : 'Failed' }}
        </p>
        <div class="flex items-center justify-between">
          <button class="fa-solid fa-pen" @click="onShowUpdateModal(todo)"></button>
          <i class="fa-solid fa-circle-check" style="color: #34e830;"></i>
        </div>
      </div>
    </div>
  </div>

  <div v-if="showUpdateModal" class="absolute w-[50%] bg-gray-400 h-[70%] left-[20%] p-3 overflow-y-scroll">
    <div class="text-right mb-6">
      <button class="p-1 px-2 bg-red-700" @click="showUpdateModal = false">X</button>
    </div>
    <form @submit.prevent="onSubmitUpdateModal">
      <div class="mb-2">
        <label class="block mb-1" for="title">Todo title</label>
        <input v-model="updateTitle" class="w-full" type="text" id="title">
      </div>
      <button class="btn block w-full">Update</button>
    </form>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
