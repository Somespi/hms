<script lang="ts">
	let { name, label, activeTab = $bindable(), onClick = () => {} } = $props();

	const isActive = $derived(activeTab === name);

	function handleClick() {
		onClick(name);
	}
</script>

<button
	class="ring-offset-background focus-visible:ring-ring inline-flex items-center justify-center rounded-sm px-3 py-1.5 text-sm font-medium whitespace-nowrap transition-all focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50 {isActive
		? 'bg-base-200 text-base-200-content'
		: 'text-base-300-content  hover:bg-base-200/50'}"
	onclick={handleClick}
	aria-selected={isActive}
	role="tab"
>
	{label}
</button>
