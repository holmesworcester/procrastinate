<template>
  <div class="hello">
    <h1>Procrastinate</h1>
    <p>The todo list for real life...</p>
    <input class="entry" type="text" ref="todoField" v-model="entry" @keydown.enter="addTodo" placeholder="...">
    <ul>
      <li v-for="todo in allTodosByDate">
        <span>
          <input class="toggle" type="checkbox" v-model="todo.done">
          <button class="due" v-if="todo.due" @click="procrastinate(todo.key)">due {{ formatDate(todo.due) }}</button>
        <span v-bind:class = "{ strikethrough: todo.done }">{{ todo.text }}</span>
      </span>
      </li>
    </ul>
  </div>
</template>

<script>
// meta todo:
// mock up a UI that makes it more self explanatory to others
// might want a +1 day
// -- MVP: i can use it --
// break things up into components to understand events / data

// show time things are due if it's the same day as today
// indicate things that are overdue or near due

var chrono = require('chrono-node')
var moment = require('moment')
var _ = require('lodash')

// localStorage persistence (from the todoMVC example)
var STORAGE_KEY = 'procrastinate'
var todoStorage = {
  fetch: function () {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    todos.forEach(function (todo, index) {
      todo.id = index
    })
    todoStorage.uid = todos.length
    return todos
  },
  save: function (todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}

export default {
  name: 'HelloWorld',
  data () {
    return {
      todos: todoStorage.fetch(),
      entry: null
    }
  },
  created: function () {
    this.$nextTick(() => this.$refs.todoField.focus())
  },
  methods: {
    addTodo: function () {
      // extract the date from the todo text
      var date = (chrono.parseDate(this.entry))
      var text = ''
      // extract the todo text from the text & date
      if (date) {
        text = (this.entry).replace((chrono.parse(this.entry))[0].text, '')
      } else {
        text = this.entry
      }
      this.todos.push({
        'text': text,
        'due': date,
        'key': this.todos.length,
        'done': false
      })
      this.entry = null
    },
    // consumes a js date and returns a string.
    formatDate: function (aDate) {
      return moment(aDate).format('MM/DD')
    },
    procrastinate: function (key) {
      this.todos[key].due = moment(this.todos[key].due).add(3, 'days')
    }
  },
  computed: {
    allTodosByDate: function () {
      // first puts done todos at the bottom. then sorts by due date
      return _.orderBy(this.todos, ['done', 'due'])
    }
  },
    // watch todos change for localStorage persistence (grabbed from todomvc project)
  watch: {
    todos: {
      handler: function (todos) {
        todoStorage.save(todos)
      },
      deep: true
    }
  }
}
</script>