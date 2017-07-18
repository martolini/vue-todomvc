<template>
  <div id="app">
    <section class="todoapp">
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo" v-model="newTodo" @keyup.enter="addTodo" autofocus autocomplete="off" placeholder="What needs to be done?">
      </header>
      <section class="main" v-show="todos.length">
        <ul class="todo-list">
          <li class="todo" v-for="todo in todos" :key="todo['.key']"
            :class="{ completed: todo.completed, editing: todo.key === editedTodo }"
          >
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input class="edit" type="text"
              v-model="todo.title"
              v-todo-focus="todo.key == editedTodo"
              @keyup.enter="doneEdit(todo)"
              @keyup.esc="doneEdit(todo)"
            >
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="todos.length" v-cloak>
        <span class="todo-count">
          {{ remaining }} {{ remaining | pluralize }} left
        </span>
      </footer>
    </section>
  </div>
</template>

<script>
import firebase from 'firebase'
const todosRef = firebase.database().ref('todos')
export default {
  name: 'app',
  data () {
    return {
      editedTodo: null,
      newTodo: '',
      loading: true
    }
  },
  firebase: {
    todos: {
      source: todosRef,
      readyCallback: function () {
        this.loading = false
      }
    }
  },
  computed: {
    remaining: function () {
      return this.todos.filter(t => !t.completed).length
    }
  },
  methods: {
    editTodo: function (todo) {
      this.editedTodo = todo.key || null
    },
    doneEdit: function (todo) {
      if (!this.editedTodo) {
        return
      }
      this.editedTodo = null
      todo.title = todo.title.trim()
    },
    addTodo: function () {
      var title = this.newTodo.trim()
      todosRef.push({
        title,
        completed: false
      })
      this.newTodo = ''
    },
    removeTodo: function (todo) {
      todosRef.child(todo['.key']).remove()
    }
  },
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  },
  filters: {
    pluralize: function (n) {
      return n === 1 ? 'task' : 'tasks'
    }
  }
}
</script>
