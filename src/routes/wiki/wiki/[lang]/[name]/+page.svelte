<script>
    import { page } from '$app/state';
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    onMount(() => {
        goto(`/wiki/${page.params.name}?lang=${page.params.lang}`);
    });
</script>