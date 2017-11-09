<template>
  <div class="hello">
    <div class="hero">
    <h1>Procrastinate</h1>
  </div>
    <div class="entryform">
    <input class="form-field" type="text" ref="todoField" v-model="entry" @keydown.enter="addTodo" placeholder="Enter todos w/ duedates, e.g. finish project by Friday">
  </div>
  <div class="todos">
    <!-- This section should get broken out into its own module as well if I show multiple lists --> 
    <ul>
      <li v-for="todo in allTodosByDate">
        <!-- This section should get broken out into its own module -->
        <button class="hyphen no-select" @click="toggleDone(todo.key)"> - </button>
        <span v-bind:class = "{ strikethrough: todo.done }">{{ todo.text }}</span>
        <span class="date-controls">
        <span v-if="todo.due && !todo.done">
          <span class="due">{{ formatDate(todo.due) }}</span>
        <span class="procrastinate">
        <button v-if="todo.due" @click="procrastinate(todo.key, 1)">+24h</button>
        <button v-if="todo.due" @click="procrastinate(todo.key, 3)">+3d</button>
        <button v-if="todo.due" @click="procrastinate(todo.key, 7)">+1w</button>
      </span>
    </span>
  </span>
      </li>
    </ul>
  </div>
</div>
</template>

<script>
// meta todo:
// will look better if I turn the checkbox into a line (like taskpaper)
// if you name a day this week that has already past, it currently sets the date to this week. it should say next week.
// use a better index for todos so that that doesn't get messed up. 
// mock up a UI that makes it more self explanatory to others
// might want a +1 day
// make a display for today / tomorrow / late so I don't need to display the date
// display the controls on rollover
// try it a few different ways
// color code it. 
// strip out "for" and "by" when it comes before dates
// maybe make an "on" and "by" distinction
// -- MVP: i can use it --
// break things up into components to understand events / data
// consider using a UI toolkit and getting comfortable with it
// show time things are due if it's the same day as today
// indicate things that are overdue or near due
// make the padding consistent for items with due dates and without so they don't shift.
// get rid of focus highlighting

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
        // no this doesn't work if you're removing todos. it needs to be some number that just keeps incrementing for ever and can never go down and gets saved when you save things. 
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
    },
    toggleDone: function (key) {
      if (this.todos[key].done) {
        this.todos[key].done = false
      } else {
        this.todos[key].done = true
      }
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