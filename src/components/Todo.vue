<script setup>
import { ref, computed, watch, onMounted } from 'vue';

let id = 0;

const newTodo = ref('');
const hideCompleted = ref(false);
const todos = ref([]);

const editingId = ref(null);
const editingText = ref('');

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

function startEditing(todo) {
	editingId.value = todo.id;
	editingText.value = todo.text;
}

function saveEdit(todo) {
	if (editingText.value.trim()) {
		todo.text = editingText.value.trim();
		editingId.value = null;
	}
}

function cancelEdit() {
	editingId.value = null;
}
</script>

<template>
	<div class="todoBox">
		<h2>Todo List</h2>

		<form @submit.prevent="addTodo" class="todoForm">
			<input v-model="newTodo" required placeholder="Add a task..." />
			<button class="btn btnAdd">Add Todo</button>
		</form>

		<ul class="todoList">
			<li v-for="todo in filteredTodos" :key="todo.id">
				<input type="checkbox" v-model="todo.done" />

				<template v-if="editingId === todo.id">
					<input v-model="editingText" class="editInput" />
					<button class="btn btnSave" @click="saveEdit(todo)">Save</button>
					<button class="btn btnCancel" @click="cancelEdit">Cancel</button>
				</template>

				<template v-else>
					<span :class="{ done: todo.done }">{{ todo.text }}</span>
					<button class="btn btnEdit" @click="startEditing(todo)">Edit</button>
					<button class="btn btnRemove" @click="removeTodo(todo)">X</button>
				</template>
			</li>
		</ul>

		<button class="btn btnHide toggleHide" @click="hideCompleted = !hideCompleted">
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
	text-align: center;
	font-size: 2em;
	margin-bottom: 1.5rem;
}

.todoForm {
	display: flex;
	flex-wrap: wrap;
	justify-content: center;
	gap: 10px;
	margin-bottom: 1rem;
	width: 100%;
	max-width: 600px;

	input {
		padding: 8px;
		font-size: 1em;
		border: 1px solid rgba(150, 150, 150, 0.5);
		border-radius: 5px;
		background: transparent;
		width: 100%;
		max-width: 300px;
		min-width: 180px;
	}

	button {
		width: 100%;
		max-width: 150px;
	}
}

.todoList {
	list-style: none;
	padding: 0;
	width: 100%;
	max-width: 600px;

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
		flex-wrap: wrap;

		input[type='checkbox'] {
			margin-right: 10px;
		}

		span {
			flex: 1;
			overflow: hidden;
			text-overflow: ellipsis;
		}

		button {
			margin-left: 5px;
			margin-top: 5px;
		}
	}
}

.editInput {
	flex: 1;
	padding: 6px;
	font-size: 1em;
	border: 1px solid #ccc;
	border-radius: 4px;
}

.done {
	text-decoration: line-through;
	color: #999;
}

.toggleHide {
	margin-top: 1rem;
	width: 100%;
	max-width: 200px;
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

.btnRemove {
	--btn-color: #e63946;
	color: #e63946;
}

.btnAdd {
	--btn-color: #2a9d8f;
	color: #2a9d8f;
}

.btnHide {
	--btn-color: #457b9d;
	color: #457b9d;
}

.btnEdit {
	--btn-color: #f4a261;
	color: #f4a261;
}

.btnSave {
	--btn-color: #2a9d8f;
	color: #2a9d8f;
}

.btnCancel {
	--btn-color: #999;
	color: #999;
}

@media (max-width: 480px) {
	.todoForm {
		flex-direction: column;
		align-items: center;

		button {
			width: 30%;
			max-width: none;
		}
	}

	.todoList li {
		align-items: flex-start;

		button {
			align-self: flex-end;
			margin-top: 5px;
		}
	}
}
</style>
