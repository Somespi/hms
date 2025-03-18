<script>
	// @ts-ignore
	// @ts-ignore
	import { onMount } from 'svelte';
	let query = '';
	// @ts-ignore
	/**
	 * @type {string | any[]}
	 */
	let results = [];
	let loading = false;
	let error = '';
	// @ts-ignore
	let timeout = null; // Store timeout ID

	function handleInput() {
		// Clear the previous timeout if it exists
		// @ts-ignore
		if (timeout) clearTimeout(timeout);

		// Set a new timeout for 1 second (1000ms)
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
				`http://9.141.41.77:8080/search?q=${encodeURIComponent(query)}&lang=ar`,
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
		} catch (err) {
			// @ts-ignore
			error = err.message;
		} finally {
			loading = false;
		}
	}
</script>

<div class="flex h-full w-full">
	<div class="bg-base-100 flex h-screen w-full flex-col items-center justify-center">
		<h1 class="p-4 text-3xl font-bold">WikiChat</h1>

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
					<a class="border-b border-gray-600 p-4 last:border-none" href={result.url}>
						<h2 class="text-lg font-bold text-white">{result.title}</h2>
						<p class="text-sm text-gray-400">{result.summary.substr(0, 100)}...

                        </p>
					</a>
				{/each}
			</div>
		{/if}
	</div>
</div>
