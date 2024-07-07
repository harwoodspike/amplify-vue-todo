<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items, isSynced }) => {
      todos.value = items
     },
  }); 
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content")
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}

  
function deleteTodo(id: string) {
  client.models.Todo.delete({ id })
}

function setDone(id: string, done: boolean) {
  client.models.Todo
    .update({ id, isDone: done })
    .then(() => {
      listTodos();
    })
}

// fetch todos when the component is mounted
 onMounted(() => {
  listTodos();
});

</script>

<template>
  <main>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <ul>
      <li 
        v-for="todo in todos" 
        :key="todo.id"
        :class="{done: todo.isDone}"
        @click="setDone(todo.id, !todo.isDone)"
      >
        <input type="checkbox" :checked="!!todo.isDone">
        {{ todo.content }}
        <button class="right" @click="deleteTodo(todo.id)">Delete</button>
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
  </main>
</template>

<style>
.done {
  background-color: lightgreen;
}
.right {
  float: right;
}
</style>