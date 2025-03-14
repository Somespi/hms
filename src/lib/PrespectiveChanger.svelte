<script lang="ts">
    export let persepictives: string[];
    export let selected: string | null = null;
</script>

<div class="flex flex-row items-center space-x-2 justify-center">
    <div>
        <div>{selected ?? 'Unselected'}</div>
    </div>
    
    <div class="dropdown relative">
        <!-- svelte-ignore a11y_no_redundant_roles -->
        <button tabindex="0" role="button" class="btn m-1">⌄</button>
        
        <ul class="dropdown-content menu bg-base-100 rounded-box z-10 w-52 p-2 shadow-sm absolute hidden">
            {#each persepictives as persepictive}
                <li>
                    <!-- svelte-ignore a11y_click_events_have_key_events -->
                    <!-- svelte-ignore a11y_no_static_element_interactions -->
                    <!-- svelte-ignore a11y_missing_attribute -->
                    <a on:click={() => selected = persepictive}>
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
