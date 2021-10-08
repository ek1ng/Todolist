<script>
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';

	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	let uid = 1;
	//todolist
	let todos = [
		{ id: uid++, done: false, trashed: false, status: 'unfinished', description: 'write some docs' },
		{ id: uid++, done: true, trashed: false, status: 'inprogress', description: 'start writing blog' },
		{ id: uid++, done: true,  trashed: true, status: 'paused', description: 'buy some milk' },
		{ id: uid++, done: false, trashed: false, status: 'inprogress', description: 'mow the lawn' },
		{ id: uid++, done: false, trashed: true, status: 'inprogress', description: 'feed the turtle' },
		{ id: uid++, done: false, trashed: false, status: 'paused', description: 'fix some bugs' },
	];

	// //新建indexdb数据库
	// var request = window.indexedDB.open('todolistdb');
	// console.log(request);
	// var db;
	// //打开数据库
	// request.onsuccess = function (event) {
  	// 	db = request.result;
	// 	console.log(db);
  	// 	console.log('数据库打开成功');
	// };
	// // 新建表并定义主键,建立索引
	// request.onupgradeneeded = function (event) {
	// 	console.log(111);
	// 	// db = event.target.result;
	// 	console.log(db);
  	// 	var objectStore;
    // 	objectStore = db.createObjectStore('todolist', { keyPath: 'id' });
	// 	objectStore.createIndex('done', 'done', { unique: false });
  	// 	objectStore.createIndex('trashed', 'trashed', { unique: false });
	// 	objectStore.createIndex('status', 'status', { unique: false });
	// 	objectStore.createIndex('description', 'description', { unique: false });
	// }

	//增加任务
	function add(input) {
		if(input.value != '') {
			const todo = {
			id: uid++,
			done: false,
			trashed: false,
			status: 'unfinished',
			description: input.value
		};
		//更新todo数组
		todos = [todo, ...todos];
		//向数据库内新增todo任务
		// var request = db.transaction(['todolist'], 'readwrite')
    	// 	.objectStore('todolist')
    	// 	.add({ id: uid++, done: false, trashed: false, status: 'unfinished', description: input.value});
  		// request.onsuccess = function (event) {
    	// 	console.log('数据写入成功');
  		// };
  		// request.onerror = function (event) {
    	// 	console.log('数据写入失败');
  		// }
		input.value = '';
		}else {
			return false;
		}
	}

	
	//删除任务
	function remove(todo) {
		todos = todos.filter(t => t !== todo);
	}

	//改变任务完成状态
	function mark(todo, done) {
		//改变checkbox选中状态
		todo.done = done;
		//将此todo移动到数组末尾
		remove(todo);
		todos = todos.concat(todo);
	}

	//改变任务是否处于垃圾箱状态
	function trashed(todo, trashed) {
		//改变是否trashed
		todo.trashed = trashed;
		todo.done = false;
		//将此todo移动到数组末尾
		remove(todo);
		todos = todos.concat(todo);
	}

	//改变状态
	function changeStatus(todo) {
		if (todo.status == 'unfinished'){
			todo.status = 'inprogress';
			console.log(todo);
		}else if (todo.status == 'inprogress'){
			todo.status = 'paused';
			console.log(todo);
		}else if (todo.status == 'paused'){
			todo.status = 'unfinished';
			console.log(todo);
		}
		todos= todos;
	}

</script>
<head>
	<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.11.0/css/all.css">
	
	<script src="https://cdn.bootcdn.net/ajax/libs/limonte-sweetalert2/11.1.0/sweetalert2.all.js"></script>
</head>

<div class='board'>
	<p class="title">To Do</p>
	<p class='subtitle'>Yesterday you said tomorrow.</p>
	<input
		placeholder="添加任务~"
		on:keydown={e => e.key === 'Enter' && e.target.value !='' && add(e.target)}
		class='addtodoinput'
	>

	<div>
		{#each todos.filter(t => !t.done && !t.trashed) as todo (todo.id)}
			<label
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}"
			>
				<input type=checkbox >
				<p 
				contenteditable = "true"
				bind:innerHTML={todo.description}
				>{todo.description}</p>
				{#if todo.status == 'unfinished'}
				<button class="unfinished" on:click={changeStatus(todo)}>
					Todo
				</button>
				{:else if todo.status == 'inprogress'}
				<button class="inprogress" on:click={changeStatus(todo)}>
					In Progress
				</button>
				{:else if todo.status == 'paused'}
				<button class="paused" on:click={changeStatus(todo)}>
					Paused
				</button>
				{/if}
				<button class="far fa-check-circle" on:click={() => mark(todo, true)}></button>
				<button on:click="{() => trashed(todo, true)}" class="fas fa-trash-alt"><i></i></button>
			</label>
		{/each}
	</div>

	<div>
		{#if todos.filter(t => t.done && !t.trashed).length > 0}
		<h2 style="font-weight:350">Done</h2>
		{#each todos.filter(t => t.done && !t.trashed) as todo (todo.id)}
			<label
				class="done"
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}"
			>
				<input type=checkbox checked>
				<p 
				contenteditable = "true"
				bind:innerHTML={todo.description}
				style="text-decoration:line-through;"
				>{todo.description}</p>
				<button class="complete" on:click={changeStatus(todo)}>
					Complete
				</button>
				<button class="far fa-calendar-times" on:click={() => mark(todo, false)}></button>
				<button on:click="{() => trashed(todo, true)}" class="fas fa-trash-alt"><i></i></button>
			</label>
		{/each}
		{/if}
	</div>

	<div>
		{#if todos.filter(t => t.trashed).length != 0}
		<h2 style="font-weight:350">Trash Bin</h2>
		{#each todos.filter(t => t.trashed) as todo (todo.id)}
			<label
				class="trashbin"
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}"
			>
				<input type=checkbox checked disabled="disabled">
				<p 
				contenteditable = "true"
				bind:innerHTML={todo.description}
				>{todo.description}</p>
				<button on:click="{() => remove(todo)}" class="fas fa-backspace"></button>
				<button on:click="{() => trashed(todo, false)}" class="fas fa-trash-restore-alt"></button>
			</label>
		{/each}
		{/if}
	</div>


</div>

<style>
	.title {
		font-weight: 400;
    	font-size: 2.5em;
		margin-bottom: 0.4em;
	}

	.subtitle {
		display : inline-block;
		font-weight: 400;
    	font-size: 1.2em;
		width: auto;
		padding: 0.2em!important;
		background: rgb(254,247,228);
	}

	.board {
		grid-template-columns: 1fr 1fr;
		grid-gap: 1em;
		max-width: 36em;
		margin: 0 auto;
	}

	.board > input {
		width: 100%;
		font-size: 1.4em;
	}

	.addtodoinput {
		margin-top: 0.4em;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
		margin: 0 0 0.5em 0;
	}

	label {
		position: relative;
		height: 2.0em;
		line-height: 1.2em;
		padding: 0.4em 2em 0.4em 0.6em;
		margin: 0 0 0.5em 0;
		border-radius: 4px;
		user-select: none;
		border: 1px solid hsl(240, 8%, 90%);
		background-color:hsl(240, 8%, 98%);
		color: #232323;
		font-size: 1.2em;
	}
	
	button {
		height: 60%;
		line-height: 60%;
	}

	input[type="checkbox"] {
		position: absolute;
		left: 0.5em;
		top: 0.6em;
		margin: 0;
		display: none;
	}

	p {
		width: 60%;
		margin: 0em;
	}

	[contenteditable] {
		margin-top:0;
		margin-bottom: 0;
		margin-right: 4.6em;
		padding: 0.4em;
		border-radius: 4px;
	}

	/* #CheckList {
        position: absolute; 
        top: 25%; 
        left: 25%; 
        width: 55%; 
        height: 55%; 
        padding: 20px; 
        border: 10px solid orange; 
        background-color: white; 
        z-index:1002; 
        overflow: auto; 
	} */

	input[type="checkbox"]:checked {
		text-decoration:line-through;
	}

	.done {
		border: 1px solid hsl(240, 8%, 90%);
		background-color:hsl(240, 8%, 98%);
	}

	.trashbin {
		border: 1px solid hsl(240, 8%, 70%);
		background-color:hsl(240, 8%, 93%);
	}

	.far.fa-check-circle {
		position: absolute;
		top: 0;
		right: 2em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}


	@-webkit-keyframes  unfinished {
		from { -webkit-box-shadow: 0 0 4px hsl(240, 8%, 48%); }
		50% { -webkit-box-shadow: 0 0 8px hsl(240, 8%, 90%); }
		to { -webkit-box-shadow: 0 0 4px hsl(240, 8%, 48%); }
	}

	.unfinished{
		position: absolute;
		top: 20%;
		width: 4em;
		right: 6em;
		height: 60%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
		color: hsl(240, 8%, 48%);
		border: 1px solid hsl(240, 8%, 48%);
	}

	.unfinished:hover {
		-webkit-animation-name: unfinished; 
		-webkit-animation-duration: 3s;
		-webkit-animation-iteration-count: infinite; 
		border: 1px solid hsl(240, 8%, 90%);
		background-color: hsl(240, 8%, 90%);
		color: white;
	}

	@-webkit-keyframes  inprogress {
		from { -webkit-box-shadow: 0 0 4px rgb(243, 176, 60); }
		50% { -webkit-box-shadow: 0 0 8px rgb(182, 118, 7); }
		to { -webkit-box-shadow: 0 0 4px rgb(243, 176, 60); }
	}

	
	.inprogress {
		position: absolute;
		top: 20%;
		width: 6em;
		right: 5em;
		height: 60%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
		text-align: center;
		color: rgb(230, 146, 36);
		border: 1px solid rgb(230, 146, 36);
	}

	.inprogress:hover {
		-webkit-animation-name: inprogress; 
		-webkit-animation-duration: 3s;
		-webkit-animation-iteration-count: infinite; 
		border: 1px solid rgb(243, 176, 80);
		background-color: rgb(243, 176, 80);
		/* color: rgb(46, 44, 42); */
		color: white;
	}

	@-webkit-keyframes  paused {
		from { -webkit-box-shadow: 0 0 4px rgb(60, 185, 243); }
		50% { -webkit-box-shadow: 0 0 8px rgb(9, 109, 148); }
		to { -webkit-box-shadow: 0 0 4px rgb(60, 185, 243); }
	}

	.paused {
		position: absolute;
		top: 20%;
		width: 4em;
		right: 6em;
		height: 60%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
		text-align: center;
		color: rgba(163, 222, 245, 0.93);
		border: 1px solid rgb(163, 222, 245);
	}

	.paused:hover {
		-webkit-animation-name: paused; 
		-webkit-animation-duration: 3s;
		-webkit-animation-iteration-count: infinite; 
		border: 1px solid rgb(60, 185, 243);
		background-color: rgb(60, 185, 243);
		/* color: #232323; */
		color: white;
	}

	@-webkit-keyframes  complete {
		from { -webkit-box-shadow: 0 0 4px rgb(159, 189, 166); }
		50% { -webkit-box-shadow: 0 0 8px rgb(79, 145, 59); }
		to { -webkit-box-shadow: 0 0 4px rgb(159, 189, 166); }
	}

	.complete {
		position: absolute;
		top: 20%;
		width: 5em;
		right: 5.5em;
		height: 60%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
		text-align: center;
		color: rgb(69, 179, 83);
		border: 1px solid rgb(69, 179, 83);
	}

	.complete:hover {
		-webkit-animation-name: complete; 
		-webkit-animation-duration: 3s;
		-webkit-animation-iteration-count: infinite; 
		border: 1px solid rgb(159, 189, 166);
		background-color: rgb(159, 189, 166);
		color: white;
	}

	.far.fa-calendar-times {
		position: absolute;
		top: 0;
		right: 2em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}
	
	.fas.fa-calendar-alt {
		position: absolute;
		top: 0;
		right: 3.8em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}

	.fas.fa-backspace {
		position: absolute;
		top: 0;
		right: 0.2em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}
	
	.fas.fa-trash-restore-alt {
		position: absolute;
		top: 0;
		right: 2em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}
	.fas.fa-trash-alt {
		position: absolute;
		top: 0;
		right: 0.2em;
		width: 2em;
		height: 100%;
		background-size: 1.4em 1.4em;
		border: none;
		background-color:transparent;
	}

</style>