<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let email = '';
  let password = '';
  let error = '';
  let loading = false;

  onMount(() => {
    if (localStorage.getItem('sa_token')) goto('/organizations');
  });

  async function handleLogin() {
    if (!email || !password) { error = 'Both fields are required'; return; }
    error = '';
    loading = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/super-admin/login`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Login failed');
      localStorage.setItem('sa_token', data.token);
      goto('/organizations');
    } catch (e) {
      error = e.message;
    } finally {
      loading = false;
    }
  }
</script>

<div class="min-h-screen flex">

  <!-- Left branding panel -->
  <div class="hidden lg:flex lg:w-1/2 bg-gradient-to-br from-violet-700 via-violet-800 to-indigo-900 flex-col justify-between p-12">
    <div>
      <div class="flex items-center gap-3">
        <div class="w-9 h-9 bg-white/20 rounded-xl flex items-center justify-center backdrop-blur">
          <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
          </svg>
        </div>
        <span class="text-white font-bold text-lg tracking-tight">Byepo</span>
      </div>
    </div>

    <div>
      <h1 class="text-4xl font-bold text-white leading-tight mb-4">
        Feature Flag<br/>Management
      </h1>
      <p class="text-violet-200 text-base leading-relaxed mb-10">
        Control feature rollouts across organizations from a single, secure admin panel.
      </p>
      <div class="space-y-4">
        {#each ['Create & manage organizations', 'Full visibility across all tenants', 'Secure role-based access'] as item}
          <div class="flex items-center gap-3">
            <div class="w-5 h-5 rounded-full bg-white/20 flex items-center justify-center flex-shrink-0">
              <svg class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
              </svg>
            </div>
            <span class="text-violet-100 text-sm">{item}</span>
          </div>
        {/each}
      </div>
    </div>

    <p class="text-violet-300 text-xs">© 2024 Byepo Technologies</p>
  </div>

  <!-- Right form panel -->
  <div class="w-full lg:w-1/2 flex items-center justify-center p-8 bg-gray-50">
    <div class="w-full max-w-sm">

      <div class="mb-8">
        <span class="inline-flex items-center gap-1.5 bg-violet-100 text-violet-700 text-xs font-semibold px-3 py-1 rounded-full mb-4">
          <svg class="w-3 h-3" fill="currentColor" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M5 9V7a5 5 0 0110 0v2a2 2 0 012 2v5a2 2 0 01-2 2H5a2 2 0 01-2-2v-5a2 2 0 012-2zm8-2v2H7V7a3 3 0 016 0z" clip-rule="evenodd"/>
          </svg>
          Super Admin Access
        </span>
        <h2 class="text-2xl font-bold text-gray-900">Welcome back</h2>
        <p class="text-gray-500 text-sm mt-1">Sign in to manage your organizations</p>
      </div>

      <form on:submit|preventDefault={handleLogin} class="space-y-5">
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700 mb-1.5">Email address</label>
          <input
            type="email" id="email"
            bind:value={email}
            placeholder="superadmin@byepo.com"
            class="w-full bg-white border border-gray-200 rounded-xl px-4 py-3 text-sm text-gray-900
                   placeholder-gray-400 shadow-sm focus:outline-none focus:ring-2 focus:ring-violet-500
                   focus:border-transparent transition"
          />
        </div>

        <div>
          <label for="password" class="block text-sm font-medium text-gray-700 mb-1.5">Password</label>
          <input
            type="password" id="password"
            bind:value={password}
            placeholder="••••••••"
            class="w-full bg-white border border-gray-200 rounded-xl px-4 py-3 text-sm text-gray-900
                   placeholder-gray-400 shadow-sm focus:outline-none focus:ring-2 focus:ring-violet-500
                   focus:border-transparent transition"
          />
        </div>

        {#if error}
          <div class="flex items-start gap-2.5 bg-red-50 border border-red-200 rounded-xl px-4 py-3">
            <svg class="w-4 h-4 text-red-500 mt-0.5 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
            <p class="text-red-700 text-sm">{error}</p>
          </div>
        {/if}

        <button
          type="submit"
          disabled={loading}
          class="w-full bg-gradient-to-r from-violet-600 to-violet-700 hover:from-violet-700 hover:to-violet-800
                 disabled:opacity-60 text-white font-semibold rounded-xl px-4 py-3 text-sm
                 shadow-sm shadow-violet-200 transition-all duration-150"
        >
          {#if loading}
            <span class="flex items-center justify-center gap-2">
              <svg class="w-4 h-4 animate-spin" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
              </svg>
              Signing in…
            </span>
          {:else}
            Sign In
          {/if}
        </button>
      </form>

    </div>
  </div>
</div>
