<svelte:options immutable={true} />

<script>
	// @ts-nocheck

	import { afterUpdate, createEventDispatcher } from 'svelte';
	import Button from '$lib//Button.svelte';

	afterUpdate(() => {
		if (autoScroll) {
			listDiv.scrollTo(0, listDivScrollHeight);
		}
		autoScroll = false;
	});

	export let todos = null;
	export let error = null;
	export let isLoading = false;
	export let disabledAdding = false;
	export let disabledItems = [];

	let prevTodos = todos;
	let inputText = '';
	let input, listDiv, autoScroll, listDivScrollHeight;
	const dispatch = createEventDispatcher();

	$: {
		autoScroll = todos && prevTodos && todos.length > prevTodos.length;
		prevTodos = todos;
	}

	export function clearInput() {
		inputText = '';
	}
	export function focusInput() {
		input.focus();
	}

	function handleAddTodo() {
		const isNotCancelled = dispatch('addTodo', { title: inputText }, { cancelable: true });

		if (isNotCancelled) {
			inputText = '';
		}
	}

	function handleRemoveTodo(id) {
		dispatch('removeTodo', { id });
	}

	function handleToggleTodo(id, value) {
		dispatch('toggleTodo', { id, value });
	}
</script>

<div class="todo-list-wrapper">
	{#if isLoading}
		<p>is loading...</p>
	{:else if error}
		<p>{error}</p>
	{:else if todos}
		<div class="todo-list" bind:this={listDiv}>
			<div bind:offsetHeight={listDivScrollHeight}>
				{#if todos.length === 0}
					<p class="no-items-text">No todos yet</p>
				{:else}
					<ul>
						{#each todos as todo, index (todo.id)}
							{@const { id, title, completed } = todo}
							<li>
								<slot {todo} {index} {handleToggleTodo}>
									<div class:completed>
										<label>
											<input
												on:input={(event) => {
													event.currentTarget.checked = completed;
													handleToggleTodo(id, !completed);
												}}
												type="checkbox"
												checked={completed}
												disabled={disabledItems.includes(id)}
											/>
											<slot name="title">{title}</slot>
										</label>
										<button
											class="remove-todo-button"
											aria-label="Remove todo: {title}"
											on:click={() => handleRemoveTodo(id)}
											disabled={disabledItems.includes(id)}>Remove</button
										>
									</div>
								</slot>
							</li>
						{/each}
					</ul>
				{/if}
			</div>
		</div>
	{/if}
	<form class="add-todo-form" on:submit|preventDefault={handleAddTodo}>
		<input
			bind:this={input}
			bind:value={inputText}
			disabled={disabledAdding || !todos}
			placeholder="New to do"
		/>
		<Button class="add-todo-button" type="submit" disabled={!inputText || disabledAdding || !todos}
			>Add</Button
		>
	</form>
</div>

<style lang="scss">
	.todo-list-wrapper {
		.todo-list {
			max-height: 150px;
			overflow: auto;
		}
		.add-todo-form {
			:global(.add-todo-button) {
				background-color: yellowgreen;
			}
		}
	}
</style>
