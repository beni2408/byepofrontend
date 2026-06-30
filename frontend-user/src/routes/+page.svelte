<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let token = '';
  let orgName = '';
  let featureKey = '';
  let result = null;
  let error = '';
  let loading = false;

  onMount(async () => {
    token = localStorage.getItem('user_token') || '';
    if (!token) { goto('/login'); return; }

    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/auth/me`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      if (res.status === 401) { logout(); return; }
      const data = await res.json();
      orgName = data.organizationName;
      localStorage.setItem('user_org_name', orgName);
    } catch {
      orgName = localStorage.getItem('user_org_name') || '';
    }
  });

  async function checkFeature() {
    if (!featureKey.trim()) { error = 'Feature key is required'; return; }
    error = '';
    result = null;
    loading = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/features/check`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${token}`
        },
        body: JSON.stringify({ key: featureKey.trim() })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Check failed');
      result = data;
    } catch (e) {
      error = e.message;
    } finally {
      loading = false;
    }
  }

  function reset() { result = null; error = ''; }

  function logout() {
    localStorage.removeItem('user_token');
    localStorage.removeItem('user_org_name');
    goto('/login');
  }
</script>

<div class="min-h-screen bg-slate-50">

  <!-- Navbar -->
  <nav class="bg-white border-b border-gray-200 px-6 flex items-center justify-between h-16 sticky top-0 z-10 shadow-sm">
    <div class="flex items-center gap-3">
      <div class="w-8 h-8 bg-gradient-to-br from-violet-600 to-violet-800 rounded-lg flex items-center justify-center shadow-sm">
        <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M3 21v-4m0 0V5a2 2 0 012-2h6.5l1 1H21l-3 6 3 6h-8.5l-1-1H5a2 2 0 00-2 2zm9-13.5V9"/>
        </svg>
      </div>
      <div>
        <span class="font-bold text-gray-900 text-sm">Byepo</span>
        <span class="text-gray-300 mx-2">·</span>
        <span class="text-gray-600 text-sm">{orgName || '…'}</span>
      </div>
    </div>
    <button
      on:click={logout}
      class="flex items-center gap-1.5 text-sm text-gray-500 hover:text-gray-900 hover:bg-gray-100 px-3 py-1.5 rounded-lg transition-colors"
    >
      <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"/>
      </svg>
      Logout
    </button>
  </nav>

  <!-- Page header -->
  <div class="bg-white border-b border-gray-100">
    <div class="max-w-lg mx-auto px-6 py-6">
      <h1 class="text-xl font-bold text-gray-900">Feature Checker</h1>
      <p class="text-sm text-gray-500 mt-0.5">Check if a feature is active for <span class="font-medium text-gray-700">{orgName || '…'}</span></p>
    </div>
  </div>

  <div class="max-w-lg mx-auto px-6 py-8 space-y-5">

    <!-- Check form -->
    <div class="bg-white rounded-2xl border border-gray-200 shadow-sm p-7">
      <h2 class="text-sm font-semibold text-gray-700 uppercase tracking-wide mb-5">Enter Feature Key</h2>

      <form on:submit|preventDefault={checkFeature} class="space-y-4">
        <div>
          <label for="featureKey" class="block text-sm font-medium text-gray-700 mb-1.5">Feature Key</label>
          <input
            type="text" id="featureKey"
            bind:value={featureKey}
            on:input={reset}
            placeholder="e.g. dark_mode, new_checkout"
            class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 text-sm font-mono
                   text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-violet-500
                   focus:border-transparent focus:bg-white transition"
          />
          <p class="text-xs text-gray-400 mt-1.5 flex items-center gap-1">
            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
            </svg>
            Results are automatically scoped to your organization
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
              Checking…
            </span>
          {:else}
            Check Feature
          {/if}
        </button>
      </form>
    </div>

    <!-- Result -->
    {#if result !== null}
      <div class="rounded-2xl overflow-hidden border {result.enabled ? 'border-green-200' : 'border-gray-200'}">
        <div class="px-7 py-5 {result.enabled
          ? 'bg-gradient-to-r from-green-500 to-emerald-500'
          : 'bg-gradient-to-r from-gray-400 to-gray-500'}">
          <div class="flex items-center justify-between">
            <span class="text-white/80 text-xs font-semibold uppercase tracking-widest font-mono">{result.key}</span>
            <span class="bg-white/20 text-white text-xs font-bold px-2.5 py-1 rounded-full">
              {result.enabled ? 'ACTIVE' : 'INACTIVE'}
            </span>
          </div>
          <p class="text-white text-3xl font-black mt-2 tracking-tight">
            {result.enabled ? 'ENABLED' : 'DISABLED'}
          </p>
        </div>
        <div class="bg-white px-7 py-4 flex items-center gap-2.5">
          <span class="w-2 h-2 rounded-full flex-shrink-0 {result.enabled ? 'bg-green-500' : 'bg-gray-400'}"></span>
          <span class="text-sm text-gray-600">
            {result.enabled
              ? 'This feature is currently active for your organization.'
              : 'This feature is currently inactive for your organization.'}
          </span>
        </div>
      </div>
    {/if}

  </div>
</div>
