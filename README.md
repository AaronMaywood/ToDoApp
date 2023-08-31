# todo

要件
- ToDo入力欄を設ける
- ToDo を入力したら key を付与し、todos に追加する
- todos リストの内容を画面に表示する
```
<script setup>
/* ToDoを追加できる */
import { ref, computed } from 'vue';

const todos = ref([]);

let idCounter = 0;
function addTodo(event){
    const newTodo = {
        key: idCounter++,
        text: event.target.value,
    };
    todos.value.push(newTodo);
}
</script>

<template>
    <h1>ToDoメモ</h1>
    <input type="text" placeholder="やりたいことを書いてください" class="text" @keyup.enter="addTodo">
    <ul>
        <li v-for="todo in todos" :key="todo.key">
            {{todo.text}}
        </li>
    </ul>
</template>

<style scoped>
.text {
    width: 600px;
    font-size: 32px;
}
</style>
```

要件
- チェックボックスで有効・無効

```
<script setup>
import { ref, computed } from 'vue';

let idCounter = 0;
const todos = ref([]);

function addTodo(event){
    const newTodo = {
        key: idCounter++,
        text: event.target.value,
        completed: false,
    };
    todos.value.push(newTodo);
}
</script>

<template>
    <h1>ToDoメモ</h1>
    <input type="text" placeholder="やりたいことを書いてください" class="text" @keyup.enter="addTodo">
    <ul>
        <li v-for="todo in todos" :key="todo.key">
            <input type="checkbox" v-model="todo.completed">
            <span :class="{completed:todo.completed}">{{todo.text}}</span>
        </li>
    </ul>
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
```


要件
- 完了したToDo を削除する

```
function removeCompleted(){
    todos.value = todos.value.filter( todo => !todo.completed );

}
<button type="button" @click="removeCompleted">完了したToDoを削除</button>
```

要件
- 追加済みToDo をダブルクリックで編集可能にする
- 編集後、Enter キーで確定

```
const editedTodo = ref();
function editTodo(todo){
    editedTodo.value = todo;
}

function doneEdit(todo){
    editedTodo.value = null;
}
</script>

<template>
    <h1>ToDoメモ</h1>
    <input type="text" placeholder="やりたいことを書いてください" class="text" @keyup.enter="addTodo">
    <ul>
        <li v-for="todo in todos" :key="todo.key">
            <input type="checkbox" v-model="todo.completed">
            <span
                v-if="editedTodo !== todo"
                :class="{completed:todo.completed}" @dblclick="editTodo(todo)">{{todo.text}}</span>
            <input type="text"
                v-if="editedTodo === todo"
                v-model="todo.text"
                @keyup.enter="doneEdit(todo)"
            >
```

要件
- 削除可能にする

```
function removeTodo(todo){
    todos.value.splice( todos.value.indexOf(todo), 1);
}

<button type="button" @click="removeTodo(todo)">削除</button>
```

要件
- 表示の状態を切り替える
	- 全て
	- 未完了
	- 完了済み

```
const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed)
};
const visibility = ref('all');
const filteredTodos = computed(() => filters[visibility.value](todos.value));

<br>
<button type="button" @click="visibility = 'all'">全て</button>
<button type="button" @click="visibility = 'active'">未完了</button>
<button type="button" @click="visibility = 'completed'">完了済み</button>
```

要件
- 未完了のToDo数を表示する
```
const remaining = computed(() => filters['active'](todos.value).length );

<span>残り{{remaining}}</span>
```

要件
- 全項目を完了済みにする「全て選択」チェックボックスを実装する
	- クリックしたら完了済みにする（チェックをつける）
	- もう一度クリックしたら未完了状態に戻る

```
function toggleAll(event){
    todos.value.forEach( todo => {
		// チェックボックスのON/OFF は .checked プロパティで取得できる
        todo.completed = event.target.checked;
    });
}

<label><input type="checkbox" @change="toggleAll">全て選択</label>
```

要件
- ToDo データをlocalstorage に保存するようにする
```
import { ref, computed, watchEffect } from 'vue';

let idCounter = 0;

// ToDoをローカルストレージに保存する
const STORAGE_KEY = 'vue-todomvc';
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
watchEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
});
```

要件
- 表示の状態を切り替える以下の選択をURLに反映し、ブックマークできるようにする
	- 全て		... URLの末尾に #/all をつけます
	- 未完了	... URLの末尾に #/active をつけます
	- 完了済み	... URLの末尾に #/completed をつけます

```
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

URLを更新する仕事をa要素にさせるため、ボタンをa要素に差し替え
<button type="button" @click="visibility = 'all'">全て</button>
<button type="button" @click="visibility = 'active'">未完了</button>
<button type="button" @click="visibility = 'completed'">完了済み</button>
↓変更
<a href="#/all" @click="visibility = 'all'">全て</a> |
<a href="#/active" @click="visibility = 'active'">未完了</a> |
<a href="#/completed" @click="visibility = 'completed'">完了済み</a>
```

