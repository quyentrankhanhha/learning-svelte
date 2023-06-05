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
		todos = [
			...todos,
			{
				id: uuidv4(),
				title: event.detail.title,
				completed: false
			}
		];
		await tick();
		todoList.clearInput();
	}

	function handleRemoveTodo(event) {
		todos = todos.filter((todo) => todo.id !== event.detail.id);
	}

	function handleToggleTodo(event) {
		todos = todos.map((todo) => {
			if (todo.id === event.detail.id) {
				return { ...todo, completed: event.detail.value };
			}

			return { ...todo };
		});
	}
</script>

<label>
	<input type="checkbox" bind:checked={showList} /> Show/ Hide List
</label>
{#if showList}
	<div style:max-width="400px" />
	<TodoList
		{todos}
		{isLoading}
		{error}
		bind:this={todoList}
		on:addTodo={handleAddTodo}
		on:removeTodo={handleRemoveTodo}
		on:toggleTodo={handleToggleTodo}
	/>
{/if}
