<script lang="ts">
	import type { ToDo } from '../types';
	import { CirclePlus, Trash } from '@lucide/svelte';
	import { onMount } from 'svelte';

	let toDos = $state<ToDo[]>([]);
	let form: HTMLFormElement;
	let inputElement: HTMLInputElement;

	const handleAddToDo = () => {
		const formData = new FormData(form);
		const description = formData.get('description') as string;
		toDos.push({ description: description, completed: false });
		form.reset();
		setToDosInLocalStorage();
		setTimeout(() => {
			inputElement.focus();
		}, 0);
	};

	const handleRemoveToDo = (toDo: { description: string; completed: boolean }) => {
		const toDoIndex = toDos.findIndex((oldToDo) => {
			return oldToDo.description === toDo.description;
		});
		toDos.splice(toDoIndex, 1);
		setToDosInLocalStorage();
	};

	const setToDosInLocalStorage = () => {
		try {
			window.localStorage.setItem('toDos', JSON.stringify(toDos));
		} catch (error) {
			console.warn('error saving todos', error);
		}
	};

	const getToDosFromLocalStorage = (): ToDo[] => {
		try {
			let savedToDos = window.localStorage.getItem('toDos');
			if (savedToDos) {
				return JSON.parse(savedToDos);
			}
		} catch (error) {
			console.warn('error getting to dos', error);
		}
		return [];
	};

	onMount(() => {
		toDos = getToDosFromLocalStorage();
	});
</script>

<div class="min-h-screen px-4 py-8">
	<div class="mx-auto max-w-2xl">
		<div class="mb-8 text-center">
			<h1 class="mb-2 text-4xl font-bold text-slate-800">TypeScript To Do List</h1>
		</div>

		<div class="mb-6 rounded-2xl bg-white p-6 shadow-lg">
			<form bind:this={form} onsubmit={handleAddToDo} class="flex gap-3">
				<input
					bind:this={inputElement}
					type="text"
					name="description"
					placeholder="What needs to be done?"
					class="flex-1 rounded-xl border border-slate-200 px-4 py-3 text-slate-700 placeholder-slate-400 transition-all duration-200 focus:border-transparent focus:ring-2 focus:ring-blue-500 focus:outline-none"
					required
				/>
				<button
					type="submit"
					class="flex items-center gap-2 rounded-xl bg-blue-500 px-6 py-3 font-medium text-white shadow-lg transition-all duration-200 hover:scale-105 hover:bg-blue-600 hover:shadow-xl active:scale-95"
				>
					<CirclePlus size={20} />
					Add
				</button>
			</form>
		</div>

		<div class="overflow-hidden rounded-2xl bg-white shadow-lg">
			{#if toDos.length === 0}
				<div class="px-6 py-12 text-center">
					<div
						class="mx-auto mb-4 flex h-16 w-16 items-center justify-center rounded-full bg-slate-100"
					>
						<CirclePlus size={24} class="text-slate-400" />
					</div>
					<h3 class="mb-2 text-lg font-medium text-slate-600">No to dos yet</h3>
					<p class="text-slate-500">Add your first to do above to get started!</p>
				</div>
			{:else}
				<ul class="divide-y divide-slate-100">
					{#each toDos as toDo, index (toDo.description)}
						<li class="group p-4 transition-colors duration-200 hover:bg-slate-50">
							<div class="flex items-center gap-4">
								<label class="flex cursor-pointer items-center">
									<input
										type="checkbox"
										name={`to-do-${index}`}
										id={`to-do-${index}`}
										bind:checked={toDo.completed}
										class="h-5 w-5 rounded border-slate-300 bg-slate-100 text-blue-600 focus:ring-2 focus:ring-blue-500"
									/>
								</label>

								<span
									class="flex-1 text-slate-700 {toDo.completed
										? 'text-slate-400 line-through'
										: ''} transition-all duration-200"
								>
									{toDo.description}
								</span>

								<button
									type="button"
									class="rounded-lg p-2 text-slate-400 opacity-0 transition-all duration-200 group-hover:opacity-100 hover:bg-red-50 hover:text-red-500"
									onclick={() => handleRemoveToDo(toDo)}
									title="Delete todo"
								>
									<Trash size={16} />
								</button>
							</div>
						</li>
					{/each}
				</ul>
			{/if}
		</div>

		{#if toDos.length > 0}
			<div class="mt-6 text-center">
				<p class="text-sm text-slate-500">
					{toDos.filter((todo) => todo.completed).length} of {toDos.length} completed
				</p>
			</div>
		{/if}
	</div>
</div>
