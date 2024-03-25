<script setup>
import { ref, computed, watchEffect } from 'vue';

// ToDoをローカルストレージに保存する
const STORAGE_KEY = 'vue-todomvc';
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
watchEffect(() => {
	localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
});

function addTodo(event){
	const newTodo = {
		key: Date.now(),
		title: event.target.value.trim(),
		completed: false,
	};
	event.target.value = '';
	todos.value.push(newTodo);
}

function removeCompleted(){
	todos.value = todos.value.filter( todo => !todo.completed );

}

const editedTodo = ref();
let beforeEditCache = '';
function editTodo(todo){
	beforeEditCache = todo.title;
	editedTodo.value = todo;
}

function cancelEdit(todo) {
	editedTodo.value = null;
	todo.title = beforeEditCache;
}

function doneEdit(todo,event){
	// done が二回実行される対策としてif文を入れている
	// はじめに@keyup.enter にてdoneEditが起動するが、doneEdit完了時に@blue が働いてもう一度doneEditが呼び出される
	if(editedTodo.value !== null){
		editedTodo.value = null;
		todo.title = todo.title.trim();
		if( todo.title === '' ) {
			removeTodo(todo);
		}
	}
}

function removeTodo(todo){
	if( todos.value.indexOf(todo) >= 0 ){
		todos.value.splice( todos.value.indexOf(todo), 1);
	}
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

function test(event){
	console.log('test',event);
}
</script>

<template>
	<section class="todoapp">
		<header>
			<h1>ToDoメモ</h1>
			<input type="text" placeholder="やりたいことを書いてください" class="new-todo" @keyup.enter="addTodo">
		</header>
		<section class="main" v-show="todos.length">
			<input id="toggle-all" class="toggle-all" type="checkbox" @change="toggleAll">
			<label for="toggle-all"><!-- 全て完了にする --></label>
			<ul class="todo-list">
				<li
					v-for="todo in filteredTodos"
					:key="todo.key"
					:class="{ completed: todo.completed, editing: todo === editedTodo }"
				>
					<div class="view">
						<input class="toggle" type="checkbox" v-model="todo.completed">
						<label
							v-if="editedTodo !== todo"
							:class="{completed:todo.completed}" @dblclick="editTodo(todo)">{{todo.title}}</label>
						<button
							class="destroy"
							type="button"
							@click="removeTodo(todo)"
						><!-- 削除 --></button>
					</div>
					<!-- @vue:mounted="({ el }) => el.focus()" -->
					<input
						type="text"
						class="edit"
						v-if="editedTodo === todo"
						v-model="todo.title"
						@vue:mounted="test"
						@blur="doneEdit(todo)"
						@keyup.enter="doneEdit(todo)"
						@keyup.escape="cancelEdit(todo)"
					>
				</li>
			</ul>
		</section>
		<footer class="footer" v-show="todos.length">
			<span class="todo-count">
				<strong>残り{{ remaining }}つ</strong>
			</span>
			<ul class="filters">
				<li>
					<a href="#/all" :class="{ selected: visibility === 'all' }">全て</a>
				</li>
				<li>
					<a href="#/active" :class="{ selected: visibility === 'active' }">未完了</a>
				</li>
				<li>
					<a href="#/completed" :class="{ selected: visibility === 'completed' }">完了済み</a>
				</li>
			</ul>
			<button
				class="clear-completed"
				@click="removeCompleted"
				v-show="todos.length >= remaining"
			>完了したToDoを削除</button>
		</footer>
	</section>
</template>

<style>
@import "./assets/style.css";
</style>
