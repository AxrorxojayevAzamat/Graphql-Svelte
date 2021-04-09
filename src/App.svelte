<script>
	import client from "./graphql-client";
	import gql from "graphql-tag"
	console.log(client)
	let todo = {
		title: '',
		completed: false
	}
	async function insertTodo() {
		const mutation = gql`
		mutation InsertTodo($title:String!, $completed:Boolean) {
			insert_Schema_Todo(objects: {title: $title, completed: $completed}) {
				affected_rows
				returning {
					id
				}
			}
		}`

		 client.mutate({  
			mutation,
			variables: {
				title: todo.title,
				completed: todo.completed
			}
		})
	}

	async function updateTodo(todo) {
		const mutation = gql`
		mutation updateTodo($id: Int!, $completed: Boolean!) {
			update_Schema_Todo(where: {id: {_eq: $id}}, _set: {completed: $completed}) {
				affected_rows
			}
		}
		`;

		client.mutate({
			mutation,
			variables: {
				id: todo.id,
				completed: todo.completed
			}
		})
	}

	const query = gql`
	subscription subscribtionTodo {
		Schema_Todo(order_by: {id: desc}) {
			completed
			id
			title
		}
	}`;

	const todos = client.subscribe({
		query
	})
</script>

<main>
	<form on:submit|preventDefault = {insertTodo}>
		<input placeholder="enter TODO title" bind:value={todo.title}>
		<input type=checkbox bind:checked={todo.completed}>Completed
		<button type="submit">Submit</button>
	</form>
</main>

{#if $todos && $todos.data}
	<table>
		<thead>
			<tr>
				<td>ID</td>
				<td>Title</td>
				<td>Status</td>
			</tr>
		</thead>
		<tbody>
			{#each $todos.data.Schema_Todo as t}
				<tr>
					<td>{t.id}</td>
					<td>{t.title}</td>
					<td>
						<label>
							<input type=checkbox bind:checked={t.completed} on:change={() => updateTodo(t)}>
							{t.completed ? "done" : "pending"}
						</label>
					</td>
				</tr>
			{/each}
		</tbody>
	</table>
{/if}

<style>
</style>