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
body {
	min-width: 230px;
	max-width: 550px;
	margin: 0 auto;
	background: #f5f5f5;
	font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
	line-height: 1.4em;
	font-weight: 300;
}

/* デフォルトのスタイルをリセット */
button {
	appearance: none;
	border: 0;
	background: none;
	font-size: 100%;
	vertical-align: baseline;
}

.todoapp {
	margin: 130px 0 40px 0;
	background: #fff;
	position: relative; /* footer::before のpositionプロパティの規準位置 */
	box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.2),
	            0 25px 50px 0 rgba(0, 0, 0, 0.1);
}

.todoapp h1 {
	position: absolute;
	top: -140px;
	width: 100%;
	text-align: center;
	color: #b83f45;
	font-size: 80px;
	font-weight: 200;
}

.new-todo,
.edit {
	padding: 6px;
	position: relative;
	width: 100%;
	margin: 0;
	font-size: 24px;
	line-height: 1.4em;
	border: 1px solid #999;
	box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
	box-sizing: border-box;
}

.new-todo {
	padding: 16px 16px 16px 60px;
	height: 65px;
	border: none;
	background: rgba(0, 0, 0, 0.003);
	box-shadow: inset 0 -2px 1px rgba(0,0,0,0.03);
}

.main {
	position: relative;
	z-index: 2;
	border-top: 1px solid #e6e6e6;
}

.toggle-all {
	/* チェックボックスを見えないように隠す */
	position: absolute;
	opacity: 0;
}

.toggle-all + label {
	font-size: 0;
	/* ボックスの大きさを指定して */
	width: 45px;
	height: 65px;
	/* そのボックスの縦横中央に表示 */
	display: flex;
	align-items: center;
	justify-content: center;
	/* 所定の位置から上にずらして表示 */
	position: absolute;
	top: -65px;
	left: -0;
}

.toggle-all + label:before {
	content: '❯';
	font-size: 22px;
	color: #949494;
	padding: 10px 27px 10px 27px;
	transform: rotate(90deg);
}

/* チェックしたら色を変える */
.toggle-all:checked + label:before {
	color: #484848;
}

.todo-list {
	margin: 0;
	padding: 0;
	list-style: none;
}

.todo-list li {
	position: relative; /* 選択するチェックボックスと削除する x アイコンの規定位置 */
	font-size: 24px;
	border-bottom: 1px solid #ededed;
}

.todo-list li:last-child {
	border-bottom: none;
}

.todo-list li.editing {
	border-bottom: none;
	padding: 0;
}

.todo-list li.editing .edit {
	display: block;
	width: calc(100% - 43px);
	padding: 12px 16px;
	margin: 0 0 0 43px;
}

.todo-list li.editing .view {
	display: none;
}

.todo-list li .toggle {
	margin: auto 0;
	appearance: none;
	position: absolute;
	top: 0;
	bottom: 0;
	/* クリック可能なエリアを作るためにサイズを指定 */
	width: 40px;
	height: auto;
}

.todo-list li .toggle {
	opacity: 0;
}

.todo-list li .toggle + label {
	/* 選択エリアを示す◯印 */
	background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A//www.w3.org/2000/svg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%23949494%22%20stroke-width%3D%223%22/%3E%3C/svg%3E');
	background-repeat: no-repeat;
	background-position: center left;
}

.todo-list li .toggle:checked + label {
	/* 選択エリアの◯に☑をつけた印 */
	background-image: url('data:image/svg+xml;utf8,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%2240%22%20height%3D%2240%22%20viewBox%3D%22-10%20-18%20100%20135%22%3E%3Ccircle%20cx%3D%2250%22%20cy%3D%2250%22%20r%3D%2250%22%20fill%3D%22none%22%20stroke%3D%22%2359A193%22%20stroke-width%3D%223%22%2F%3E%3Cpath%20fill%3D%22%233EA390%22%20d%3D%22M72%2025L42%2071%2027%2056l-4%204%2020%2020%2034-52z%22%2F%3E%3C%2Fsvg%3E');
}

.todo-list li label {
	word-break: break-all;
	padding: 15px 15px 15px 60px;
	display: block;
	line-height: 1.2;
	transition: color 0.4s;
	font-weight: 400;
	color: #484848;
}

.todo-list li.completed label {
	color: #949494;
	text-decoration: line-through;
}

.todo-list li .destroy {
	display: none;
	position: absolute;
	top: 0;
	right: 10px;
	bottom: 0;
	width: 40px;
	height: 40px;
	margin: auto 0;
	font-size: 30px;
	color: #949494;
	transition: color 0.2s ease-out;
}

.todo-list li .destroy:hover,
.todo-list li .destroy:focus {
	color: #C18585;
}

.todo-list li .destroy:after {
	content: '×';
	display: block;
	height: 100%;
	line-height: 1.1;
}

.todo-list li:hover .destroy {
	display: block;
}

.todo-list li .edit {
	display: none;
}

.todo-list li.editing:last-child {
	margin-bottom: -1px;
}

.footer {
	padding: 10px 15px;
	height: 20px;
	text-align: center;
	font-size: 15px;
	border-top: 1px solid #e6e6e6;
	display: flex;
	justify-content: space-between;
}

/* box-shadowを駆使して階段状の影 */
.footer:before {
	content: '';
	position: absolute;
	right: 0;
	left: 0;
	bottom: 0;
	height: 50px;
	box-shadow: 0 1px 1px rgba(0, 0, 0, 0.2),
	            0 8px 0 -3px #f6f6f6,
	            0 9px 1px -3px rgba(0, 0, 0, 0.2),
	            0 16px 0 -6px #f6f6f6,
	            0 17px 2px -6px rgba(0, 0, 0, 0.2);
}

.todo-count strong {
	font-weight: 300;
}

.filters {
	margin: 0;
	padding: 0;
	list-style: none;
	/* 中央寄せ */
	position: absolute;
	right: 0;
	left: 0;
	/* 子要素を横並びで中央寄せ */
	display: flex;
	justify-content: center;
}

.filters li a {
	color: inherit;
	margin: 3px;
	padding: 3px 7px;
	text-decoration: none;
	border: 1px solid transparent;
	border-radius: 3px;
}

.filters li a:hover {
	border-color: #DB7676;
}

.filters li a.selected {
	border-color: #CE4646;
}

.clear-completed,
.clear-completed:active {
	line-height: 19px;
	cursor: pointer;
	/* 左隣のulがposition:absolute; でこの要素の上に重なっているため、重なり順をコントロールするのに relative を指定している */
	position: relative;
}

.clear-completed:hover {
	text-decoration: underline;
}

@media (max-width: 430px) {
	.footer {
		height: 50px;
	}

	.filters {
		bottom: 10px;
	}
}

/* クリックして選択した箇所に赤枠をつけて目立たせる */
:focus,
.toggle:focus + label,
.toggle-all:focus + label {
	box-shadow: 0 0 2px 2px #CF7D7D;
	outline: 0;
}
</style>
