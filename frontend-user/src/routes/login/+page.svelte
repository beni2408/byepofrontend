<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let email = '';
  let password = '';
  let error = '';
  let loading = false;

  onMount(() => {
    if (localStorage.getItem('user_token')) goto('/');
  });

  async function handleLogin() {
    if (!email || !password) { error = 'Both fields are required'; return; }
    error = '';
    loading = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/auth/login`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Login failed');
      if (data.role !== 'end_user') {
        throw new Error('This portal is for end users only');
      }
      localStorage.setItem('user_token', data.token);
      localStorage.setItem('user_org_name', data.organizationName);
      goto('/');
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
    <div class="flex items-center gap-3">
      <div class="w-9 h-9 bg-white/20 rounded-xl flex items-center justify-center backdrop-blur">
        <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M3 21v-4m0 0V5a2 2 0 012-2h6.5l1 1H21l-3 6 3 6h-8.5l-1-1H5a2 2 0 00-2 2zm9-13.5V9"/>
        </svg>
      </div>
      <span class="text-white font-bold text-lg tracking-tight">Byepo</span>
    </div>

    <div>
      <h1 class="text-4xl font-bold text-white leading-tight mb-4">
        Feature Flag<br/>Checker
      </h1>
      <p class="text-violet-200 text-base leading-relaxed mb-10">
        Sign in to check whether any feature is active for your organization.
      </p>
      <div class="space-y-4">
        {#each ['Scoped to your organization automatically', 'Instant enabled / disabled response', 'Secure — no org ID guessing needed'] as item}
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

    <p class="text-violet-300 text-xs">© 2026 Byepo Technologies</p>
  </div>

  <!-- Right form panel -->
  <div class="w-full lg:w-1/2 flex items-center justify-center p-8 bg-gray-50">
    <div class="w-full max-w-sm">

      <div class="mb-8">
        <span class="inline-flex items-center gap-1.5 bg-violet-100 text-violet-700 text-xs font-semibold px-3 py-1 rounded-full mb-4">
          <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/>
          </svg>
          End User
        </span>
        <h2 class="text-2xl font-bold text-gray-900">Sign in</h2>
        <p class="text-gray-500 text-sm mt-1">Access your feature flag checker</p>
      </div>

      <form on:submit|preventDefault={handleLogin} class="space-y-5">
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700 mb-1.5">Email address</label>
          <input
            type="email" id="email"
            bind:value={email}
            placeholder="you@yourorg.com"
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

      <p class="text-center text-sm text-gray-500 mt-6">
        No account yet?
        <a href="/signup" class="text-violet-700 font-semibold hover:underline">Create one</a>
      </p>

    </div>
  </div>
</div>
