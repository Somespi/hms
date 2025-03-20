<script lang="ts">
	import PrespectiveChanger from '$lib/PrespectiveChanger.svelte';
	import { onMount } from 'svelte';
	import { Send } from 'lucide-svelte';
	import axios from 'axios';
	import { page } from '$app/state';
	import { get } from 'svelte/store';
	import { marked } from 'marked';
	let { children } = $props();

	let selectedPersepictive: string | null = $state('Unbiased');

	let messageInput: HTMLInputElement;

	let isSidebarOpen: boolean = $state(true);
	let isResizing: boolean = $state(false);
	let sidebar;

	const wikiName: string = page.params.wiki_name;
	let wikiLanguage: string;

	let chatHistory: {
		role: string;
		content: string;
	}[] = $state([]);

	let chatId: string = $state('');

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

	const getChatId = async () => {
		// check local storage for chat id for this wiki name
		const chatId = localStorage.getItem(`chatId-${wikiName}-${wikiLanguage}`);
		if (chatId) {
			return chatId;
		} else {
			const id = crypto.randomUUID();
			localStorage.setItem(`chatId-${wikiName}-${wikiLanguage}`, id);
			return id;
		}
	};

	const askChatbot = (message: string) => {
		// add thinking message to chat history until we get a response
		axios
			.post('https://wikiapi.abdlmutii.me/chat', {
				lang: wikiLanguage,
				message: message,
				model: selectedPersepictive,
				uuid: chatId,
				wiki: wikiName
			})
			.then((res) => {
				console.log(res);
				chatHistory[chatHistory.length - 1].content = res.data.message;
				chatHistory = [...chatHistory];
			});
		chatHistory = [
			...chatHistory,
			{ role: 'user', content: message },
			{ role: 'ai', content: 'Thinking...' }
		];
		messageInput.value = '';
	};

	const getChatHistory = async () => {
		const response = await axios.get(`https://wikiapi.abdlmutii.me/messages/${chatId}`);
		chatHistory = response.data.messages.map((message: any) => {
			return {
				role: message.role === 'User' ? 'user' : 'ai',
				content: message.content
			};
		});
	};

	const getPromptSuggestions = async () => {
		try {
			const response = await axios.post('https://wikiapi.abdlmutii.me/prompts', {
				lang: wikiLanguage,
				wiki: wikiName
			});
			console.log(response);
			return response.data;
		} catch (error) {
			console.error(error);
		}
	};

	onMount(async () => {
		document.addEventListener('mousemove', handleMouseMove);
		document.addEventListener('mouseup', handleMouseUp);
		document.getElementById('chat-history')!.style.height =
			document.getElementById('chat-history')!.clientHeight + 'px';
		document.getElementById('chat-history')!.classList.remove('flex-1');

		const urlParams = new URLSearchParams(window.location.search);
		wikiLanguage = urlParams.get('lang') || 'en';

		let data = await getPromptSuggestions();

		getChatId().then((id) => {
			chatId = id;
			getChatHistory();
		});


		console.log(data);

		function findLastQuestions(obj: any): string[] {
			if (!obj) return [];
			while (obj?.questions) obj = obj.questions;
			return Array.isArray(obj) && obj.length >= 3 ? obj.slice(0, 3) : [];
		}

		if (data && data.questions) {
			prompts = findLastQuestions(data);
		}
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
							<div class="chat-bubble chat-bubble-neutral text-sm">
								{@html marked(message.content)}
							</div>
						</div>
					{/if}
				{/each}
			</div>

			{#if prompts.length > 0 && chatHistory.length === 0}
				<div class="no-scrollbar flex flex-row gap-2 overflow-x-auto">
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
