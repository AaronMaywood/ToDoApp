<script setup>
import { ref, computed, watchEffect } from 'vue';

let idCounter = 0;

// ToDoをローカルストレージに保存する
const STORAGE_KEY = 'vue-todomvc';
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
watchEffect(() => {
	localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
});

function addTodo(event){
	const newTodo = {
		key: idCounter++,
		text: event.target.value,
		completed: false,
	};
	todos.value.push(newTodo);
}

function removeCompleted(){
	todos.value = todos.value.filter( todo => !todo.completed );

}

const editedTodo = ref();
function editTodo(todo){
	editedTodo.value = todo;
}

function doneEdit(todo){
	editedTodo.value = null;
}

function removeTodo(todo){
	todos.value.splice( todos.value.indexOf(todo), 1);
}


const filters = {
	all: (todos) => todos,
	active: (todos) => todos.filter((todo) => !todo.completed),
	completed: (todos) => todos.filter((todo) => todo.completed)
};
const visibility = ref('all');
const filteredTodos = computed(() => filters[visibility.value](todos.value));
const remaining = computed(() => filters['active'](todos.value).length );

function toggleAll(event){
	todos.value.forEach( todo => {
		todo.completed = event.target.checked;
	});
}

// handle routing
window.addEventListener('hashchange', onHashChange);
onHashChange();
function onHashChange() {
	const route = window.location.hash.replace('#/', '')
	if (filters[route]) {
		visibility.value = route;
	} else {
		window.location.hash = '';
		visibility.value = 'all';
	}
}

</script>

<template>
	<h1>ToDoメモ</h1>
	<input type="text" placeholder="やりたいことを書いてください" class="text" @keyup.enter="addTodo">

	<label><input type="checkbox" @change="toggleAll">全て選択</label>
	<br>
	<ul>
		<li v-for="todo in filteredTodos" :key="todo.key">
			<input type="checkbox" v-model="todo.completed">
			<span
				v-if="editedTodo !== todo"
				:class="{completed:todo.completed}" @dblclick="editTodo(todo)">{{todo.text}}</span>
			<input type="text"
				v-if="editedTodo === todo"
				v-model="todo.text"
				@keyup.enter="doneEdit(todo)"
			>
			<button type="button"
				@click="removeTodo(todo)"
			>削除</button>
		</li>
	</ul>
	<button type="button" @click="removeCompleted">完了したToDoを削除</button>
	<br>
	<span>残り{{remaining}}</span>
	<a href="#/all" @click="visibility = 'all'">全て</a> | 
	<a href="#/active" @click="visibility = 'active'">未完了</a> | 
	<a href="#/completed" @click="visibility = 'completed'">完了済み</a>
</template>

<style scoped>
.text {
	width: 600px;
	font-size: 32px;
}

li {
	font-size: 32px;
}

.completed {
	text-decoration: line-through;
}
</style>
