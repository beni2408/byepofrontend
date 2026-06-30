<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  onMount(() => {
    if (localStorage.getItem('sa_token')) goto('/organizations');
    else goto('/login');
  });
</script>
