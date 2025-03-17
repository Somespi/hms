<script lang="ts">
	import PrespectiveChanger from '$lib/PrespectiveChanger.svelte';
	import { onMount } from 'svelte';
	import '../app.css';
	import { Send } from 'lucide-svelte';
	let { children } = $props();

	let selectedPersepictive: string | null = $state('Unbiased');

	let isSidebarOpen: boolean = $state(true);
	let isResizing: boolean = $state(false);
	let sidebar;

	let chatHistory: {
		role: string;
		content: string;
	}[] = $state([]);

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

	const addMessage = (message: string) => {
		chatHistory = [...chatHistory, { role: 'user', content: message }];
		console.log(chatHistory);
	};

	onMount(() => {
		document.addEventListener('mousemove', handleMouseMove);
		document.addEventListener('mouseup', handleMouseUp);
		addMessage('What\'s so special about Egypt?');
		document.getElementById('chat-history')!.style.height = document.getElementById('chat-history')!.clientHeight + 'px';
		document.getElementById('chat-history')!.classList.remove('flex-1');
		chatHistory = [...chatHistory, { role: 'ai', content: `Egypt is super special because it's home to one of the oldest and most amazing civilizations ever! Think giant pyramids, mysterious pharaohs, and ancient temples along the Nile River. It's like stepping into a real-life history book, with incredible stories and sights that have fascinated people for thousands of years.` }];
	});
</script>

<div class="flex h-full w-full">
	<div
		class="bg-base-100 flex h-screen md:w-1/4 flex-col border-r w-full"
		bind:this={sidebar}
		class:hidden={!isSidebarOpen}
	>
		<!-- svelte-ignore a11y_click_events_have_key_events -->
		<!-- svelte-ignore a11y_interactive_supports_focus -->
		<!-- svelte-ignore event_directive_deprecated -->
		<div class="flex h-14 items-center border-b px-4 justify-start">
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
			<div id="chat-history" class="flex-1 overflow-y-auto " >
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
							<div class="chat-bubble text-sm  chat-bubble-neutral">{message.content}</div>
						</div>
					{/if}
				{/each}
			</div>

			<div class="border-t p-4">
				<div class="flex items-center gap-2">
				<input type="text" placeholder="Type here..." class="flex-1 rounded-md border border-input bg-background px-3 py-2 text-sm ring-0 input focus-visible:outline-none" />
				<button class="inline-flex items-center justify-center rounded-md h-10 w-10 bg-primary text-primary-content hover:bg-primary/90">
						<Send class="h-4 w-4 " />
				</button>
			</div>
			</div>
		</div>
	</div>

	<div class="max-h-screen flex-1 overflow-y-auto leading-relaxed bg-base-200">
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
