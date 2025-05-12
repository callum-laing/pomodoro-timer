<script setup>
import { ref, computed, watch, onMounted } from 'vue';

let id = 0;

const newTodo = ref('');
const hideCompleted = ref(false);
const todos = ref([]);

onMounted(() => {
	const savedTodos = localStorage.getItem('todos');
	const savedHideCompleted = localStorage.getItem('hideCompleted');

	if (savedTodos) {
		todos.value = JSON.parse(savedTodos);
		id = Math.max(0, ...todos.value.map((t) => t.id)) + 1;
	}

	if (savedHideCompleted !== null) {
		hideCompleted.value = JSON.parse(savedHideCompleted);
	}
});

watch(
	todos,
	(newTodos) => {
		localStorage.setItem('todos', JSON.stringify(newTodos));
	},
	{ deep: true }
);

watch(hideCompleted, (newValue) => {
	localStorage.setItem('hideCompleted', JSON.stringify(newValue));
});

const filteredTodos = computed(() => {
	return hideCompleted.value ? todos.value.filter((t) => !t.done) : todos.value;
});

function addTodo() {
	if (newTodo.value.trim()) {
		todos.value.push({ id: id++, text: newTodo.value.trim(), done: false });
		newTodo.value = '';
	}
}

function removeTodo(todo) {
	todos.value = todos.value.filter((t) => t !== todo);
}
</script>

<template>
	<div class="todoBox">
		<h2><span>Todo List</span></h2>
		<form @submit.prevent="addTodo" class="todoForm">
			<input v-model="newTodo" required placeholder="Add a task..." />
			<button class="btn btnWork">Add Todo</button>
		</form>

		<ul class="todoList">
			<li v-for="todo in filteredTodos" :key="todo.id">
				<input type="checkbox" v-model="todo.done" />
				<span :class="{ done: todo.done }">{{ todo.text }}</span>
				<button class="btn btnShort" @click="removeTodo(todo)">X</button>
			</li>
		</ul>

		<button class="btn btnLong" @click="hideCompleted = !hideCompleted">
			{{ hideCompleted ? 'Show all' : 'Hide completed' }}
		</button>
	</div>
</template>

<style scoped lang="scss">
.todoBox {
	display: flex;
	flex-direction: column;
	align-items: center;
	font-size: 1.2em;
	margin-top: 5rem;
	padding: 1rem;
}

h2 {
	width: 50%;
	text-align: center;
	border-bottom: 1px solid rgba(250, 250, 250, 0.5);
	line-height: 0.1em;
	margin: 10px 0 50px;
	font-size: 2em;
}

h2 span {
	background: rgba(20, 20, 20);
	padding: 0 10px;
}

.todoForm {
	display: flex;
	flex-wrap: wrap;
	justify-content: center;
	gap: 10px;
	margin-bottom: 1rem;

	input {
		padding: 8px;
		font-size: 1em;
		border: 1px solid rgba(150, 150, 150, 0.5);
		border-radius: 5px;
		min-width: 200px;
		background: transparent;
	}
}

.todoList {
	list-style: none;
	padding: 0;
	width: 100%;
	max-width: 400px;

	li {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 10px;
		background: transparent;
		padding: 8px 10px;
		border-radius: 5px;
		border: 1px solid rgba(150, 150, 150, 0.5);
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

		input[type='checkbox'] {
			margin-right: 10px;
		}

		span {
			flex: 1;
			overflow: hidden;
			text-overflow: ellipsis;
		}
	}
}

.done {
	text-decoration: line-through;
	color: #999;
}

.btn {
	border: 2px solid var(--btn-color);
	background-color: transparent;
	font-weight: bold;
	border-radius: 5px;
	padding: 5px 12px;
	cursor: pointer;
	transition: all 0.2s ease-in-out;
}

.btn:hover {
	box-shadow: 0 0 6px var(--btn-color);
}

.btnWork {
	--btn-color: #e63946;
	color: #e63946;
}

.btnShort {
	--btn-color: #2a9d8f;
	color: #2a9d8f;
}

.btnLong {
	--btn-color: #457b9d;
	color: #457b9d;
}

@media (max-width: 480px) {
	.todoForm {
		flex-direction: column;

		input,
		button {
			width: 100%;
		}
	}

	.todoList li {
		flex-direction: column;
		align-items: flex-start;

		button {
			align-self: flex-end;
			margin-top: 5px;
		}
	}
}
</style>
