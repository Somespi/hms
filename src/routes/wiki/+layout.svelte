<script lang="ts">
	import PrespectiveChanger from '$lib/PrespectiveChanger.svelte';
	import { onMount } from 'svelte';
	import { Send } from 'lucide-svelte';
	import axios from 'axios';
	import { page } from '$app/stores';
	import { get } from 'svelte/store';
	let { children } = $props();

	let selectedPersepictive: string | null = $state('Unbiased');

	let messageInput: HTMLInputElement;

	let isSidebarOpen: boolean = $state(true);
	let isResizing: boolean = $state(false);
	let sidebar;

	let wikiName: string;
	let wikiLanguage: string;

	let chatHistory: {
		role: string;
		content: string;
	}[] = $state([]);

	let prompts: string[] = $state([]);

	const handleMouseDown = () => {
		isResizing = true;
	};

	const handleMouseMove = (event: MouseEvent) => {
		if (!isResizing) return;

		const currentWidth = sidebar!.offsetWidth + event.movementX;

		if (currentWidth < 25) {
			isSidebarOpen = false;
			sidebar!.style.width = `25px`;
		} else {
			isSidebarOpen = true;
			sidebar!.style.width = `${currentWidth}px`;
		}

		if (!isSidebarOpen && event.movementX < 0) {
			sidebar!.style.width = `40px`;
		}

		console.log(event.movementX, sidebar!.style.width);
	};

	const handleMouseUp = () => {
		isResizing = false;
	};

	const askChatbot = (message: string) => {
		// add thinking message to chat history until we get a response
		axios
			.post('https://wikiless.serveo.net/chat', {
				lang: wikiLanguage,
				message: message,
				model: selectedPersepictive,
				uuid: '123e4567-e89b-12d3-a456-426614174000',
				wiki: wikiName
			})
			.then((res) => {
				console.log(res);
				// remove thinking message and add response
				chatHistory = chatHistory.filter((msg) => msg.content !== 'Thinking...');
				chatHistory = [...chatHistory, { role: 'ai', content: res.data.message }];
			});
		chatHistory = [
			...chatHistory,
			{ role: 'user', content: message },
			{ role: 'ai', content: 'Thinking...' }
		];
		messageInput.value = '';
	};

	const getPromptSuggestions = () => {
		return axios
			.post('https://wikiless.serveo.net/prompts', {
				lang: wikiLanguage,
				wiki: wikiName
			})
			.then((response) => {
				console.log(response);
				return response.data;
			})
			.catch((error) => {
				console.error(error);
			});
	};

	onMount(() => {
		document.addEventListener('mousemove', handleMouseMove);
		document.addEventListener('mouseup', handleMouseUp);
		document.getElementById('chat-history')!.style.height =
			document.getElementById('chat-history')!.clientHeight + 'px';
		document.getElementById('chat-history')!.classList.remove('flex-1');

		const url = new URL(get(page).url);

		// Extract the wiki name from the pathname
		const pathSegments = url.pathname.split('/');
		wikiName =
			pathSegments.length > 2
				? decodeURIComponent(pathSegments[2]).replaceAll('_', ' ')
				: 'Unknown Wiki';

		// Extract the language from query parameters
		const params = new URLSearchParams(url.search);
		wikiLanguage = params.get('lang') || 'en';

		getPromptSuggestions().then((data: { questions: string[] }) => {
			function findLastQuestions(obj: any): string[] {
				while (obj?.questions) obj = obj.questions;
				return Array.isArray(obj) && obj.length >= 3 ? obj.slice(0, 3) : [];
			}

			if (data.questions) {
				prompts = findLastQuestions(data);
			}
			return prompts;
		});
	});
</script>

<div class="flex h-full w-full">
	<div
		class="bg-base-100 flex h-screen w-full flex-col border-r md:w-1/4"
		bind:this={sidebar}
		class:hidden={!isSidebarOpen}
	>
		<!-- svelte-ignore a11y_click_events_have_key_events -->
		<!-- svelte-ignore a11y_interactive_supports_focus -->
		<!-- svelte-ignore event_directive_deprecated -->
		<div class="flex h-14 items-center justify-start border-b px-4">
			<PrespectiveChanger
				persepictives={[
					'Conservative',
					'Liberal',
					'Green',
					'Historic',
					'Progressive',
					'Society',
					'Eastern',
					'Western'
				]}
				bind:selected={selectedPersepictive}
			/>
		</div>

		<div class="flex flex-1 flex-col space-y-2 p-4">
			<div id="chat-history" class="flex-1 overflow-y-auto">
				{#each chatHistory as message}
					{#if message.role === 'user'}
						<div class="chat chat-start">
							<div class="chat-header">You</div>
							<div class="chat-bubble chat-bubble-primary text-sm">{message.content}</div>
						</div>
					{:else}
						<div class="chat chat-end">
							<div class="chat-header">
								{selectedPersepictive} Bot
							</div>
							<div class="chat-bubble chat-bubble-neutral text-sm">{message.content}</div>
						</div>
					{/if}
				{/each}
			</div>

			{#if prompts.length > 0}
				<div class="flex flex-row gap-2">
					{#each prompts as prompt}
						<button
							class="btn btn-ghost btn-xs border-input border border-gray-700"
							onclick={() => askChatbot(prompt)}
						>
							{prompt}
						</button>
					{/each}
				</div>
			{/if}

			<div class="border-t p-4">
				<div class="flex items-center gap-2">
					<input
						type="text"
						placeholder="Type here..."
						class="border-input bg-background input flex-1 rounded-md border px-3 py-2 text-sm ring-0 focus-visible:outline-none"
						id="message"
						bind:this={messageInput}
					/>
					<button
						class="bg-primary text-primary-content hover:bg-primary/90 btn btn-ghost btn-xs inline-flex h-10 w-10 items-center justify-center rounded-md"
						onclick={() => askChatbot(messageInput.value)}
					>
						<Send class="h-4 w-4 " />
					</button>
				</div>
			</div>
		</div>
	</div>

	<div class="bg-base-200 max-h-screen flex-1 overflow-y-auto leading-relaxed">
		<div class="flex flex-row">
			<div class="hidden md:block">
				<!-- svelte-ignore a11y_no_static_element_interactions -->
				<!-- svelte-ignore event_directive_deprecated -->
				<div
					class="bg-base-200 fixed h-screen w-4 cursor-ew-resize"
					onmousedown={handleMouseDown}
				></div>
			</div>
			<div class="ml-5 flex-1 p-5">
				{@render children()}
			</div>
		</div>
	</div>
</div>
