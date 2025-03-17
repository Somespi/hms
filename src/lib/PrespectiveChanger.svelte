<script lang="ts">
    import { ChevronDown } from 'lucide-svelte';
	export let persepictives: string[];
	export let selected: string | null = null;
</script>

<div class="flex flex-row items-center justify-start ">
	<span class="font-medium">{selected ?? 'Unbiased'}</span>
	<div class="dropdown relative">
		<!-- svelte-ignore a11y_no_redundant_roles -->
		<button tabindex="0" role="button" class="btn m-1 btn-sm btn-ghost">
            <ChevronDown class="h-4 w-4" />
        </button>

		<ul
			class="dropdown-content menu bg-base-100 rounded-box absolute z-10 hidden w-52 p-2 shadow-sm"
		>
			{#each persepictives as persepictive}
				<li>
					<!-- svelte-ignore a11y_click_events_have_key_events -->
					<!-- svelte-ignore a11y_no_static_element_interactions -->
					<!-- svelte-ignore a11y_missing_attribute -->
					<a onclick={() => (selected = persepictive)}>
						{persepictive}
					</a>
				</li>
			{/each}
		</ul>
	</div>
</div>

<style>
	.dropdown:hover .dropdown-content {
		display: block;
	}
</style>
