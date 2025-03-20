<script lang='ts'>
	import { onMount } from 'svelte';
	let query = '';
	let results: { title: string; url: string; summary: string }[] = [];
	let loading = false;
	let error = '';
	let timeout: string | number | NodeJS.Timeout | null | undefined = null; // Store timeout ID

	function handleInput() {
		// Clear the previous timeout if it exists
		if (timeout) clearTimeout(timeout);
		timeout = setTimeout(() => {
			search();
		}, 1000);
	}

	async function search() {
		if (!query.trim()) {
			results = [];
			return;
		}

		loading = true;
		error = '';

		try {
			const res = await fetch(
				`https://became-geometry-operating-berry.trycloudflare.com/search?q=${encodeURIComponent(query)}`,
				{
					headers: {
						Accept: 'application/json',
                        'Allow-Control-Allow-Origin': '*',
					},
				}
			);

			if (!res.ok) {
				throw new Error('Failed to fetch search results');
			}

			results = await res.json();
			console.log(results);
		} catch (err: Error | any) {
			error = err.message;
		} finally {
			loading = false;
		}
	}
</script>

<div class="flex h-full w-full">
	<div class="bg-base-100 flex h-screen w-full flex-col items-center justify-center">
		<h1 class="p-4 text-3xl font-bold">WikiLess</h1>

		<label class="input flex w-full max-w-xl items-center gap-2">
			<svg class="h-[1em] opacity-50" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
				<g
					stroke-linejoin="round"
					stroke-linecap="round"
					stroke-width="2.5"
					fill="none"
					stroke="currentColor"
				>
					<circle cx="11" cy="11" r="8"></circle>
					<path d="m21 21-4.3-4.3"></path>
				</g>
			</svg>
			<input
				type="search"
				bind:value={query}
				on:input={handleInput}
				required
				placeholder="Search"
			/>
		</label>

		<!-- Loading Indicator -->
		{#if loading}
			<p class="mt-4 text-gray-500">Searching...</p>
		{/if}

		<!-- Error Message -->
		{#if error}
			<p class="mt-4 text-red-500">{error}</p>
		{/if}

		<!-- Search Results -->
		{#if results.length > 0}
			<div class="mt-1 w-full max-w-xl rounded-lg bg-gray-700 shadow-md flex flex-col gap-2">
				{#each results as result}
					<a class="border-b border-gray-600 p-4 last:border-none" href="/wiki/{result.title}?lang=en">
						<h2 class="text-lg font-bold text-white">{result.title}</h2>
						<p class="text-sm text-gray-400">{result.summary.slice(0, 100)}...

                        </p>
					</a>
				{/each}
			</div>
		{/if}
	</div>
</div>
