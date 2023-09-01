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
        title: event.target.value,
    };
    todos.value.push(newTodo);
}
</script>

<template>
    <h1>ToDoメモ</h1>
    <input type="text" placeholder="やりたいことを書いてください" class="text" @keyup.enter="addTodo">
    <ul>
        <li v-for="todo in todos" :key="todo.key">
            {{todo.title}}
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

--------------------------------------------------------------------------------
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
        title: event.target.value,
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
            <span :class="{completed:todo.completed}">{{todo.title}}</span>
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


--------------------------------------------------------------------------------
要件
- 完了したToDo を削除する

```
function removeCompleted(){
    todos.value = todos.value.filter( todo => !todo.completed );

}
<button type="button" @click="removeCompleted">完了したToDoを削除</button>
```

--------------------------------------------------------------------------------
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
                :class="{completed:todo.completed}" @dblclick="editTodo(todo)">{{todo.title}}</span>
            <input type="text"
                v-if="editedTodo === todo"
                v-model="todo.title"
                @keyup.enter="doneEdit(todo)"
            >
```

--------------------------------------------------------------------------------
要件
- 削除可能にする

```
function removeTodo(todo){
    todos.value.splice( todos.value.indexOf(todo), 1);
}

<button type="button" @click="removeTodo(todo)">削除</button>
```
--------------------------------------------------------------------------------

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

--------------------------------------------------------------------------------
要件
- 未完了のToDo数を表示する
```
const remaining = computed(() => filters['active'](todos.value).length );

<span>残り{{remaining}}</span>
```

--------------------------------------------------------------------------------
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

--------------------------------------------------------------------------------
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
--------------------------------------------------------------------------------

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
↓変更 URLからvisibility.value を書き換えるようになったのでここでは@clickは不要、差し替え
<a href="#/all">全て</a> |
<a href="#/active">未完了</a> |
<a href="#/completed">完了済み</a>

--------------------------------------------------------------------------------
要件
- 編集中に編集の枠外をクリックすると編集中モードから抜ける
	https://developer.mozilla.org/ja/docs/Web/API/Element/blur_event
	blur イベントは、要素がフォーカスを失ったときに発生します。

```
 97                     <input
 98                         type="text"
 99                         class="edit"
							@vue:mounted="({ el }) => el.focus()"	追加 ダブルクリックするとこの要素にフォーカスが行く。これを入れないと、もう一度クリックしないと選択されない。フォーカス済みにして初めて@blur（や、次の要件である[Esc]入力が）効くようになる
100                         v-if="editedTodo === todo"
101                         v-model="todo.title"
102                         @blur="doneEdit(todo)"				追加
103                         @keyup.enter="doneEdit(todo)"
104                     >
```

- 編集中に[ESC]キーで編集をキャンセル（編集中の内容は元通りになる）
TODO
- [esc] を押すと、@keyup.escape より前に @blur が効く（効き、doneEditされて入力値が反映され、キャンセルされない)ように思える
	→これはEndeavour OS の場合の挙動で、Windowsで見たら問題なかった
		公式の
		https://ja.vuejs.org/examples/#todomvc
		でも、[Esc]でキャンセルできていなかった（doneの動きになる）
- cancel のあとに done が来るのは避けられないのでdone に対策をする

```
            @keyup.escape="cancelEdit(todo)"

- @vue:mounted="({el})=> とは？
	→Vue.jsのどのドキュメントに掲載されているか？
		https://ja.vuejs.org/api/composition-api-lifecycle.html#onmounted
			- そのコンポーネント自身の DOM ツリーが作成され、親コンテナーに挿入された時。
		el は$elとは関係がないな
		https://ja.vuejs.org/api/component-instance.html#el
		https://zenn.dev/ojk/books/intro-to-vue/viewer/vue-if#mounted-%E3%81%A8-unmounted
			@vue はディレクティブ
				↓公式ドキュメントのディレクティブにはv-vueというのはのっていないなあ
				https://ja.vuejs.org/api/built-in-directives.html			
			@vue は v-on ではないかという指摘
			「頭に @ が付いていることから察するに、これらは v-on イベントの一種という扱いだと思います。v-on と使い方も同じです。」
			https://ja.vuejs.org/api/built-in-directives.html#v-on
				これらを指しているのかなあ？
					<!-- 動的イベント -->
					<button v-on:[event]="doThis"></button>	←これか
					<!-- 動的イベントの省略記法 -->
					<button @[event]="doThis"></button>
				イベントの一覧は？
				https://ja.vuejs.org/guide/components/events.html#emitting-and-listening-to-events
					ここには情報はない

ドキュメントに見当たらないので、使い方を調べておく
```
                    <input
                        type="text"
                        class="edit"
                        v-if="editedTodo === todo"
                        v-model="todo.title"
                        @vue:mounted="test"			// ダミー関数に差し替え
                        @blur="doneEdit(todo)"
                        @keyup.enter="doneEdit(todo)"
                        @keyup.escape="cancelEdit(todo)"
                    >

function test(event){
    console.log('test',event);	// {type: 'input', el: input.edit, ...}
}
```
test()に渡されたイベントオブジェクトの中に、確かに el: input.edit が入っているのがわかった。



--------------------------------------------------------------------------------
要件
- 入力したToDoの前後の空白文字を消す
```
function addTodo(event){
    const newTodo = {
        key: idCounter++,
        title: event.target.value.trim(),	// 変更
        completed: false,
    };
    todos.value.push(newTodo);
}
function doneEdit(todo) {
    editedTodo.value = null
    todo.title = todo.title.trim()		// 追加
}
```

--------------------------------------------------------------------------------
要件
- 項目を追加したあとに、入力欄を空っぽにする
```
function addTodo(event){
    const newTodo = {
        key: idCounter++,
        title: event.target.value.trim(),
        completed: false,
    };
    event.target.value = '';	// 追加
    todos.value.push(newTodo);
}
```

--------------------------------------------------------------------------------
要件
- 編集中のToDoを空文字にしたときには、このToDOを消す
```
function doneEdit(todo) {
	editedTodo.value = null
	todo.title = todo.title.trim()
	if (!todo.title) removeTodo(todo)		// 追加
}
```
→落とし穴あり！上のコードでは、周辺のToDoまで削除してしまう
以下、その調査と対策
```
調査のために、$event を渡し、@blueと@keyup.enter のどちらのイベントが先に読み込まれているかを調査

                    <input
                        type="text"
                        class="edit"
                        v-if="editedTodo === todo"
                        v-model="todo.title"
                        @vue:mounted="({ el }) => el.focus()"
                        @blur="doneEdit(todo, $event)"
                        @keyup.enter="doneEdit(todo, $event)"
                        @keyup.escape="cancelEdit(todo)"
                    >



function doneEdit(todo,event){
console.log('done edit', todo,event);		// event を出力して調査
// done が二回実行される対策（特にremoveのために)
//  @blur="doneEdit(todo)"
//  @keyup.enter="doneEdit(todo)"
// はじめに@keyup.enter にてdoneEditが起動するが、doneEdit完了時に@blue が働いてもう一度doneEditが呼び出される
	editedTodo.value = null;
	todo.title = todo.title.trim();
	if( todo.title === '' ) {
console.log('go remove', todo);
		removeTodo(todo);
	}
}
```
その結果、想像通り @keyup.enter → @blue の順番でdoneが二段階で呼び出されるのがわかった
https://i.gyazo.com/2b15bf7f75bc4f84003f237717a3b491.png

    if(editedTodo.value !== null){
    }


なお、doneEdit によって二回呼び出されるremoveTodo() でなぜターゲットToDoだけじゃなく、その周りのToDo まで消してしまうかだが、 .indexOf() の返り値が -1 (該当なし）のときに削除しないようになっていなかったからだ
```
function removeTodo(todo){
    todos.value.splice( todos.value.indexOf(todo), 1);
}
↓修正
function removeTodo(todo){
    if( todos.value.indexOf(todo) >= 0 ){
        todos.value.splice( todos.value.indexOf(todo), 1);
    }
}
```
doneEdit が二回呼び出される対策は、removeTodo をこのように書くだけでもOK.

結果として以下のコードになった

```
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
```



--------------------------------------------------------------------------------
要件
- スタイルシートを実装する

--------------------------------------------------------------------------------
要件
- ToDoが無いときには一覧とフッターを非表示にする
	<section class="main" v-show="todos.length">
	<footer class="footer" v-show="todos.length">

--------------------------------------------------------------------------------
要件
- 「完了したToDoを削除」は、何か一つでも削除したToDoがあるときのみに表示する

```
<button
	class="clear-completed"
	@click="removeCompleted"
	v-show="todos.length >= remaining"	// 追加
>完了したToDoを削除</button>
```
v-showの式は以下でもOK
	v-show="filters['completed'](todos).length >= 1"


テンプレートの中なので、todos.value ではなくtodos にすること。（todos.value にするとエラーで動作しなくなる）


