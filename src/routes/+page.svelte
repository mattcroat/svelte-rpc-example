<script lang="ts">
	import { addTodo, deleteTodo, getTime, getTodos, toggleTodo } from './todos.remote'

	/*
		- `getTodos` does a remote function call on the client
			using `fetch` but is invoked directly on the server
		- SSR is not implemented yet in the async branch
	*/
	const todos = getTodos()
</script>

<main>
	<h1>Todo App</h1>

	<!-- prerendering example -->
	<p>Time: {await getTime()}</p>

	<!-- using enhance to customize how the form is progressively enhanced -->
	<form
		{...addTodo.enhance(async ({ data, form, submit }) => {
			// get form data
			const text = data.get('text')!.toString().trim()

			// optimistic UI update
			await submit().updates(
				getTodos().withOverride((todos) => {
					return [...todos, { id: '0', text, done: false }]
				})
			)

			// clear form input
			form.reset()
		})}
	>
		<input type="text" name="text" placeholder="Add todo" autocomplete="off" />
		<button type="submit">Add</button>
		{#if addTodo.error}
			<p class="error">{addTodo.error.message}</p>
		{/if}
	</form>

	<ul>
		<!-- async svelte ❤️ -->
		{#each await todos as todo}
			<!-- useful when you have multiple forms that use the same remote form action for reuse -->
			{@const remove = deleteTodo.for(todo.id)}

			<li>
				<label>
					<!-- this should be a form but we want to showcase using commands -->
					<input
						type="checkbox"
						checked={todo.done}
						onchange={async () => {
							// optimistic UI update
							await toggleTodo(todo.id).updates(
								getTodos().withOverride((todos) => {
									return todos.map((t) => (t.id === todo.id ? { ...t, done: !t.done } : t))
								})
							)
						}}
					/>
					<span class={{ done: todo.done }}>{todo.text}</span>
				</label>

				<!-- using enhance to customize how the form is progressively enhanced -->
				<form
					{...remove.enhance(async ({ submit }) => {
						// optimistic UI update
						await submit().updates(
							getTodos().withOverride((todos) => {
								return todos.filter((t) => t.id !== todo.id)
							})
						)
					})}
				>
					<!-- this seems to have bugs 🐛  -->
					{#if remove.error}
						<span class="error">{remove.error.message}</span>
					{/if}
					<button name="id" value={todo.id}>Delete</button>
				</form>
			</li>
		{/each}
	</ul>
</main>

<style>
	.done {
		text-decoration: line-through;
	}

	.error {
		color: red;
	}
</style>
