<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let token = '';
  let orgName = '';
  let flags = [];
  let newKey = '';
  let error = '';
  let success = '';
  let loadingFlags = false;
  let creating = false;

  onMount(async () => {
    token = localStorage.getItem('admin_token') || '';
    if (!token) { goto('/login'); return; }

    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/auth/me`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      if (res.status === 401) { logout(); return; }
      const data = await res.json();
      orgName = data.organizationName;
      localStorage.setItem('admin_org_name', orgName);
    } catch {
      orgName = localStorage.getItem('admin_org_name') || '';
    }

    await loadFlags();
  });

  async function loadFlags() {
    loadingFlags = true;
    error = '';
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/flags`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      if (res.status === 401) { logout(); return; }
      flags = await res.json();
    } catch {
      error = 'Failed to load flags';
    } finally {
      loadingFlags = false;
    }
  }

  async function createFlag() {
    if (!newKey.trim()) { error = 'Flag key is required'; return; }
    error = ''; success = '';
    creating = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/flags`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
        body: JSON.stringify({ key: newKey.trim() })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Failed to create');
      success = `Flag "${data.key}" created`;
      newKey = '';
      await loadFlags();
    } catch (e) {
      error = e.message;
    } finally {
      creating = false;
    }
  }

  async function toggleFlag(flag) {
    flag.enabled = !flag.enabled;
    flags = flags;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/flags/${flag._id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
        body: JSON.stringify({ enabled: flag.enabled })
      });
      if (!res.ok) {
        flag.enabled = !flag.enabled;
        flags = flags;
        const data = await res.json();
        error = data.error || 'Toggle failed';
      }
    } catch {
      flag.enabled = !flag.enabled;
      flags = flags;
      error = 'Network error';
    }
  }

  async function deleteFlag(flag) {
    if (!confirm(`Delete flag "${flag.key}"? This cannot be undone.`)) return;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/flags/${flag._id}`, {
        method: 'DELETE',
        headers: { Authorization: `Bearer ${token}` }
      });
      if (!res.ok) { error = 'Failed to delete'; return; }
      flags = flags.filter(f => f._id !== flag._id);
      success = `Flag "${flag.key}" deleted`;
    } catch {
      error = 'Network error';
    }
  }

  function logout() {
    localStorage.removeItem('admin_token');
    localStorage.removeItem('admin_org_id');
    localStorage.removeItem('admin_org_name');
    goto('/login');
  }

  $: enabledCount = flags.filter(f => f.enabled).length;
  $: disabledCount = flags.filter(f => !f.enabled).length;
</script>

<div class="min-h-screen bg-slate-50">

  <!-- Navbar -->
  <nav class="bg-white border-b border-gray-200 px-6 flex items-center justify-between h-16 sticky top-0 z-10 shadow-sm">
    <div class="flex items-center gap-3">
      <div class="w-8 h-8 bg-gradient-to-br from-violet-600 to-violet-800 rounded-lg flex items-center justify-center shadow-sm">
        <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
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

  <!-- Page header with stats -->
  <div class="bg-white border-b border-gray-100">
    <div class="max-w-4xl mx-auto px-6 py-6">
      <div class="flex items-start justify-between">
        <div>
          <h1 class="text-xl font-bold text-gray-900">Feature Flags</h1>
          <p class="text-sm text-gray-500 mt-0.5">Managing flags for <span class="font-medium text-gray-700">{orgName || '…'}</span></p>
        </div>
        <div class="flex items-center gap-3">
          <div class="text-center bg-green-50 border border-green-100 rounded-xl px-4 py-2 min-w-[64px]">
            <p class="text-xl font-bold text-green-600">{enabledCount}</p>
            <p class="text-xs text-green-500 font-medium">On</p>
          </div>
          <div class="text-center bg-gray-50 border border-gray-200 rounded-xl px-4 py-2 min-w-[64px]">
            <p class="text-xl font-bold text-gray-400">{disabledCount}</p>
            <p class="text-xs text-gray-400 font-medium">Off</p>
          </div>
          <div class="text-center bg-violet-50 border border-violet-100 rounded-xl px-4 py-2 min-w-[64px]">
            <p class="text-xl font-bold text-violet-700">{flags.length}</p>
            <p class="text-xs text-violet-500 font-medium">Total</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="max-w-4xl mx-auto px-6 py-8 space-y-6">

    <!-- Feedback messages -->
    {#if error}
      <div class="flex items-center gap-3 bg-red-50 border border-red-200 rounded-xl px-4 py-3">
        <svg class="w-4 h-4 text-red-500 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <p class="text-red-700 text-sm">{error}</p>
        <button on:click={() => error = ''} class="ml-auto text-red-400 hover:text-red-600">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>
    {/if}
    {#if success}
      <div class="flex items-center gap-3 bg-green-50 border border-green-200 rounded-xl px-4 py-3">
        <svg class="w-4 h-4 text-green-500 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
        </svg>
        <p class="text-green-700 text-sm">{success}</p>
        <button on:click={() => success = ''} class="ml-auto text-green-400 hover:text-green-600">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>
    {/if}

    <!-- Create flag -->
    <div class="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
      <h2 class="text-sm font-semibold text-gray-700 uppercase tracking-wide mb-4">New Feature Flag</h2>
      <form on:submit|preventDefault={createFlag} class="flex gap-3">
        <input
          bind:value={newKey}
          placeholder="e.g. dark_mode, new_checkout, beta_dashboard"
          class="flex-1 bg-gray-50 border border-gray-200 rounded-xl px-4 py-2.5 text-sm font-mono
                 text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-violet-500
                 focus:border-transparent focus:bg-white transition"
        />
        <button
          type="submit"
          disabled={creating}
          class="bg-gradient-to-r from-violet-600 to-violet-700 hover:from-violet-700 hover:to-violet-800
                 disabled:opacity-60 text-white font-semibold rounded-xl px-5 py-2.5 text-sm
                 shadow-sm transition-all whitespace-nowrap flex items-center gap-2"
        >
          {#if creating}
            <svg class="w-4 h-4 animate-spin" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
            </svg>
            Creating…
          {:else}
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
            </svg>
            Create Flag
          {/if}
        </button>
      </form>
    </div>

    <!-- Flags list -->
    <div class="bg-white rounded-2xl border border-gray-200 shadow-sm overflow-hidden">
      <div class="px-6 py-4 border-b border-gray-100 flex items-center justify-between">
        <h2 class="font-semibold text-gray-900">All Flags</h2>
        <button
          on:click={loadFlags}
          class="flex items-center gap-1.5 text-xs text-gray-400 hover:text-gray-700 hover:bg-gray-100 px-2.5 py-1.5 rounded-lg transition-colors"
        >
          <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"/>
          </svg>
          Refresh
        </button>
      </div>

      {#if loadingFlags}
        <div class="flex items-center justify-center gap-3 py-16 text-gray-400">
          <svg class="w-5 h-5 animate-spin" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
          </svg>
          <span class="text-sm">Loading flags…</span>
        </div>
      {:else if flags.length === 0}
        <div class="flex flex-col items-center justify-center py-16 text-center">
          <div class="w-14 h-14 bg-violet-50 rounded-2xl flex items-center justify-center mb-4">
            <svg class="w-7 h-7 text-violet-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M3 21v-4m0 0V5a2 2 0 012-2h6.5l1 1H21l-3 6 3 6h-8.5l-1-1H5a2 2 0 00-2 2zm9-13.5V9"/>
            </svg>
          </div>
          <p class="text-sm font-semibold text-gray-700">No flags yet</p>
          <p class="text-xs text-gray-400 mt-1">Create your first feature flag above</p>
        </div>
      {:else}
        <ul class="divide-y divide-gray-50">
          {#each flags as flag (flag._id)}
            <li class="px-6 py-4 flex items-center justify-between gap-4 hover:bg-gray-50/60 transition-colors group">
              <div class="flex items-center gap-3 min-w-0">
                <div class="w-8 h-8 rounded-lg {flag.enabled ? 'bg-green-100' : 'bg-gray-100'} flex items-center justify-center flex-shrink-0 transition-colors">
                  <svg class="w-4 h-4 {flag.enabled ? 'text-green-600' : 'text-gray-400'} transition-colors" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 21v-4m0 0V5a2 2 0 012-2h6.5l1 1H21l-3 6 3 6h-8.5l-1-1H5a2 2 0 00-2 2zm9-13.5V9"/>
                  </svg>
                </div>
                <div class="min-w-0">
                  <p class="text-sm font-mono font-semibold text-gray-900 truncate">{flag.key}</p>
                  <span class="inline-flex items-center gap-1 mt-0.5 text-xs font-medium {flag.enabled ? 'text-green-600' : 'text-gray-400'}">
                    <span class="w-1.5 h-1.5 rounded-full {flag.enabled ? 'bg-green-500' : 'bg-gray-300'} inline-block"></span>
                    {flag.enabled ? 'Enabled' : 'Disabled'}
                  </span>
                </div>
              </div>

              <div class="flex items-center gap-3 flex-shrink-0">
                <button
                  on:click={() => toggleFlag(flag)}
                  class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors
                         focus:outline-none focus:ring-2 focus:ring-violet-500 focus:ring-offset-2"
                  class:bg-violet-600={flag.enabled}
                  class:bg-gray-200={!flag.enabled}
                  aria-label="Toggle {flag.key}"
                >
                  <span
                    class="inline-block h-4 w-4 transform rounded-full bg-white shadow-sm transition-transform duration-200"
                    class:translate-x-6={flag.enabled}
                    class:translate-x-1={!flag.enabled}
                  ></span>
                </button>

                <button
                  on:click={() => deleteFlag(flag)}
                  class="w-8 h-8 flex items-center justify-center rounded-lg text-gray-300
                         hover:text-red-500 hover:bg-red-50 opacity-0 group-hover:opacity-100
                         transition-all duration-150"
                  aria-label="Delete {flag.key}"
                >
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                  </svg>
                </button>
              </div>
            </li>
          {/each}
        </ul>
      {/if}
    </div>

  </div>
</div>
