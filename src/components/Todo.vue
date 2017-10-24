<template>
	<span>
		<input class="toggle" type="checkbox" @click="toggleDone">
		<button class="due" v-if="due" @click="procrastinate">due {{ formatDate(due) }}</button>
		<span v-bind:class = "{ strikethrough: done }">{{ text }}</span>
	</span>
</template>

<script>

var moment = require('moment')

export default {
	name: 'Todo',
 // I pass in text, due date, and whether it's done or not
 // will want to use v-bind with these possibly?
 props: ['text', 'due', 'done'],
 methods: {
 // emit a "procrastinate" event
 procrastinate: function () {
 	this.$emit('procrastinate')
 },
  // okay now this is different because I can't just bind the checkbox status to done. I guess I just emit a toggle event whenver it changes.
  toggleDone: function () {
  	this.$emit('toggleDone')
  },
  // consumes a js date and returns a string.
  formatDate: function (aDate) {
  	return moment(aDate).format('MM/DD')
  },
}
}
</script>
