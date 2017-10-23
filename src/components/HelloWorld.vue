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
// might want a +1 day
// deploy to gh-pages
// -- MVP: i can use it --
// break things up into components to understand events / data
// make it more self explanatory to others
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


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

ul {
  text-align: left;
  margin: 2em 15vw;
}

li {
  list-style-type: none;
}

h1 {
  margin: 1em 0 0 0;
  font-size: 7vw;
  text-transform: uppercase;
}

.due {
  font-size: .8em;
  color: white;
  background: darkgrey;
  border: none;
  border-radius: 4px;
  padding: .2em .5em;
}

p { 
  margin: 0;
  padding: 0 0 3em 0;
  font-size: 1em;
}

.entry {
  padding: 0 0.3em;
  height: 1.7em;
  width: 59vw;
  font-size: 1.5em;
  border-radius: 5px;
  box-shadow: none;
}

.strikethrough {
  text-decoration: line-through;
}

</style>
