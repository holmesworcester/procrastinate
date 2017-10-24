<template>
  <div class="hello">
    <div class="hero">
    <h1>Procrastinate</h1>
    <p>The todo list for real life...</p>
  </div>
    <div class="entryform">
    <input class="form-field" type="text" ref="todoField" v-model="entry" @keydown.enter="addTodo" placeholder="Enter your todos here, with due dates!">
    <p class="example">E.g. "Buy bread today", "Get presents December 24th", "Finish prototype by next Friday"</p>
  </div>
  <div class="todos">
    <!-- This section should get broken out into its own module as well if I show multiple lists --> 
    <h2 v-if="todos.length">Todo:</h2>
    <ul>
      <li v-for="todo in allTodosByDate">
        <!-- This section should get broken out into its own module -->
        <span>
          <input class="toggle" type="checkbox" v-model="todo.done">
          <span class="due" v-if="todo.due">{{ formatDate(todo.due) }}</span>
        <span v-bind:class = "{ strikethrough: todo.done }">{{ todo.text }}</span>
        <button v-if="todo.due" class="procrastinate" @click="procrastinate(todo.key, 1)">+1 day</button>
        <button v-if="todo.due" class="procrastinate" @click="procrastinate(todo.key, 3)">+3 days</button>
      </span>
      </li>
    </ul>
  </div>
</div>
</template>

<script>
// meta todo:
// mock up a UI that makes it more self explanatory to others
// might want a +1 day
// make a display for today / tomorrow / late
// color code it. 
// strip out "for" and "by" when it comes before dates
// -- MVP: i can use it --
// break things up into components to understand events / data
// consider using a UI toolkit and getting comfortable with it
// show time things are due if it's the same day as today
// indicate things that are overdue or near due

var chrono = require('chrono-node')
var moment = require('moment')
var _ = require('lodash')

// import component
// import Todo from './Todo.vue'

// localStorage persistence (from the todoMVC example)
var STORAGE_KEY = 'procrastinate2'
var todoStorage = {
  fetch: function () {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    // I need to understand how this works. It might not work as I expect it to.
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
  // register component
  // components: {
  //  Todo
  // },
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
        // gives every todo a unique key (I think this works)
        'key': this.todos.length,
        'done': false
      })
      this.entry = null
    },
    // consumes a js date and returns a string.
    formatDate: function (aDate) {
      return moment(aDate).format('MM/DD')
    },
    // bumps a todo's due date forward 3 days
    procrastinate: function (key, days) {
      this.todos[key].due = moment(this.todos[key].due).add(days, 'days')
    }
  },
  computed: {
    // sorts the todos to first put "done" ones at the bottom and then sort everything else by due date.
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