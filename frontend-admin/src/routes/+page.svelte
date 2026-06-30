<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  onMount(() => {
    if (localStorage.getItem('admin_token')) goto('/flags');
    else goto('/login');
  });
</script>
