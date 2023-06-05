<script>
	// @ts-nocheck

	import TodoList from '$lib/TodoList.svelte';
	import { onMount, tick } from 'svelte';
	import { v4 as uuidv4 } from 'uuid';

	let todos = null;
	let todoList;
	let showList = true;
	let error = null;
	let isLoading = false;
	let isAdding = false;
	let disabledItems = [];

	onMount(() => {
		loadTodos();
	});

	async function loadTodos() {
		isLoading = true;
		await fetch('https://jsonplaceholder.typicode.com/todos?_limit=10').then(async (response) => {
			if (response.ok) {
				todos = await response.json();
			} else {
				error = 'An error occured while fetching the todos';
			}
		});
		isLoading = false;
	}

	async function handleAddTodo(event) {
		event.preventDefault();
		isAdding = true;

		await fetch('https://jsonplaceholder.typicode.com/todos', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json; charset=UTF-8' },
			body: JSON.stringify({
				title: event.detail.title,
				completed: false
			})
		}).then(async (response) => {
			if (response.ok) {
				const todo = await response.json();
				todos = [...todos, { ...todo, id: uuidv4() }];
				todoList.clearInput();
			} else {
				alert('An error occured while adding the todo');
			}
		});

		isAdding = false;
		await tick();
		todoList.focusInput();
	}

	async function handleRemoveTodo(event) {
		const id = event.detail.id;
		if (disabledItems.includes(id)) return;
		disabledItems = [...disabledItems, id];

		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'DELETE'
		}).then(async (response) => {
			if (response.ok) {
				todos = todos.filter((todo) => todo.id !== id);
			} else {
				alert('An error occured while removing the todo');
			}
		});
		disabledItems = disabledItems.filter((itemId) => itemId !== id);
	}

	async function handleToggleTodo(event) {
		const id = event.detail.id;
		const value = event.detail.value;
		if (disabledItems.includes(id)) return;
		disabledItems = [...disabledItems, id];

		await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
			method: 'PATCH',
			body: JSON.stringify({ completed: value }),
			headers: { 'Content-Type': 'application/json; charset=UTF-8' }
		}).then(async (response) => {
			if (response.ok) {
				const updatedTodo = await response.json();
				todos = todos.map((todo) => {
					if (todo.id === id) {
						return updatedTodo;
					}

					return { ...todo };
				});
			} else {
				alert('An error occured while removing the todo');
			}
		});
		disabledItems = disabledItems.filter((itemId) => itemId !== id);
	}
</script>

<label>
	<input type="checkbox" bind:checked={showList} /> Show/ Hide List
</label>
{#if showList}
	<div>
		<TodoList
			{todos}
			{isLoading}
			{error}
			{disabledItems}
			disabledAdding={isAdding}
			bind:this={todoList}
			on:addTodo={handleAddTodo}
			on:removeTodo={handleRemoveTodo}
			on:toggleTodo={handleToggleTodo}
			let:index
			let:todo
		>
			<span slot="title">{index + 1} - {todo.title}</span>
		</TodoList>
	</div>
{/if}
