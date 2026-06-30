<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let email = '';
  let password = '';
  let organizationId = '';
  let error = '';
  let loading = false;

  onMount(() => {
    if (localStorage.getItem('user_token')) goto('/');
  });

  async function handleSignup() {
    if (!email || !password || !organizationId.trim()) {
      error = 'All fields are required';
      return;
    }
    error = '';
    loading = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/auth/user-signup`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password, organizationId: organizationId.trim() })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Signup failed');
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
        Check your<br/>features
      </h1>
      <p class="text-violet-200 text-base leading-relaxed mb-10">
        Sign up to instantly check whether any feature is active for your organization.
      </p>
      <div class="bg-white/10 backdrop-blur rounded-2xl p-5 border border-white/20">
        <p class="text-xs font-semibold text-violet-200 uppercase tracking-wide mb-3">What you get</p>
        <div class="space-y-3">
          {#each [
            ['1', 'Register with your Organization ID'],
            ['2', 'Log in securely with your account'],
            ['3', 'Check any feature flag with one click']
          ] as [num, text]}
            <div class="flex items-start gap-3">
              <span class="w-5 h-5 rounded-full bg-white/20 text-white text-xs font-bold flex items-center justify-center flex-shrink-0">{num}</span>
              <span class="text-violet-100 text-sm">{text}</span>
            </div>
          {/each}
        </div>
      </div>
    </div>

    <p class="text-violet-300 text-xs">© 2024 Byepo Technologies</p>
  </div>

  <!-- Right form panel -->
  <div class="w-full lg:w-1/2 flex items-center justify-center p-8 bg-gray-50">
    <div class="w-full max-w-sm">

      <div class="mb-8">
        <span class="inline-flex items-center gap-1.5 bg-violet-100 text-violet-700 text-xs font-semibold px-3 py-1 rounded-full mb-4">
          <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"/>
          </svg>
          End User
        </span>
        <h2 class="text-2xl font-bold text-gray-900">Create account</h2>
        <p class="text-gray-500 text-sm mt-1">Join your organization to check feature flags</p>
      </div>

      <form on:submit|preventDefault={handleSignup} class="space-y-5">
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

        <div>
          <label for="orgId" class="block text-sm font-medium text-gray-700 mb-1.5">Organization ID</label>
          <input
            type="text" id="orgId"
            bind:value={organizationId}
            placeholder="64abc123def456..."
            class="w-full bg-white border border-gray-200 rounded-xl px-4 py-3 text-sm font-mono text-gray-900
                   placeholder-gray-400 shadow-sm focus:outline-none focus:ring-2 focus:ring-violet-500
                   focus:border-transparent transition"
          />
          <p class="text-xs text-gray-400 mt-1.5 flex items-center gap-1">
            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
            Provided by your organization administrator
          </p>
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
              Creating account…
            </span>
          {:else}
            Create Account
          {/if}
        </button>
      </form>

      <p class="text-center text-sm text-gray-500 mt-6">
        Already have an account?
        <a href="/login" class="text-violet-700 font-semibold hover:underline">Sign in</a>
      </p>

    </div>
  </div>
</div>
