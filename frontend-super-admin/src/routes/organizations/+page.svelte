<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { PUBLIC_API_URL } from '$env/static/public';

  let token = '';
  let orgs = [];
  let newName = '';
  let error = '';
  let success = '';
  let loadingOrgs = false;
  let creating = false;
  let editingId = null;
  let editingName = '';
  let saving = false;

  onMount(async () => {
    token = localStorage.getItem('sa_token') || '';
    if (!token) { goto('/login'); return; }
    await loadOrgs();
  });

  async function loadOrgs() {
    loadingOrgs = true;
    error = '';
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/super-admin/organizations`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      if (res.status === 401) { logout(); return; }
      orgs = await res.json();
    } catch {
      error = 'Failed to load organizations';
    } finally {
      loadingOrgs = false;
    }
  }

  async function createOrg() {
    if (!newName.trim()) { error = 'Name is required'; return; }
    error = ''; success = '';
    creating = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/super-admin/organizations`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
        body: JSON.stringify({ name: newName.trim() })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Failed to create');
      success = `"${data.name}" created successfully`;
      newName = '';
      await loadOrgs();
    } catch (e) {
      error = e.message;
    } finally {
      creating = false;
    }
  }

  function startEdit(org) {
    editingId = org._id;
    editingName = org.name;
    error = ''; success = '';
  }

  function cancelEdit() {
    editingId = null;
    editingName = '';
  }

  async function saveEdit(org) {
    if (!editingName.trim()) { error = 'Name cannot be empty'; return; }
    if (editingName.trim() === org.name) { cancelEdit(); return; }
    error = ''; success = '';
    saving = true;
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/super-admin/organizations/${org._id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
        body: JSON.stringify({ name: editingName.trim() })
      });
      const data = await res.json();
      if (!res.ok) throw new Error(data.error || 'Failed to update');
      success = `Renamed to "${data.name}"`;
      cancelEdit();
      await loadOrgs();
    } catch (e) {
      error = e.message;
    } finally {
      saving = false;
    }
  }

  async function deleteOrg(org) {
    if (!confirm(`Delete "${org.name}"? This cannot be undone.`)) return;
    error = ''; success = '';
    try {
      const res = await fetch(`${PUBLIC_API_URL}/api/super-admin/organizations/${org._id}`, {
        method: 'DELETE',
        headers: { Authorization: `Bearer ${token}` }
      });
      if (!res.ok) {
        const data = await res.json();
        throw new Error(data.error || 'Failed to delete');
      }
      success = `"${org.name}" deleted`;
      await loadOrgs();
    } catch (e) {
      error = e.message;
    }
  }

  function logout() {
    localStorage.removeItem('sa_token');
    goto('/login');
  }

  function formatDate(d) {
    return new Date(d).toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' });
  }
</script>

<div class="min-h-screen bg-slate-50">

  <!-- Navbar -->
  <nav class="bg-white border-b border-gray-200 px-6 py-0 flex items-center justify-between h-16 sticky top-0 z-10 shadow-sm">
    <div class="flex items-center gap-3">
      <div class="w-8 h-8 bg-gradient-to-br from-violet-600 to-violet-800 rounded-lg flex items-center justify-center shadow-sm">
        <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
        </svg>
      </div>
      <div>
        <span class="font-bold text-gray-900 text-sm">Byepo</span>
        <span class="text-gray-300 mx-2">·</span>
        <span class="text-gray-600 text-sm">Super Admin</span>
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
    <div class="max-w-5xl mx-auto px-6 py-6 flex items-center justify-between">
      <div>
        <h1 class="text-xl font-bold text-gray-900">Organizations</h1>
        <p class="text-sm text-gray-500 mt-0.5">Manage tenants across your platform</p>
      </div>
      <div class="flex items-center gap-2 bg-violet-50 border border-violet-100 rounded-xl px-4 py-2">
        <span class="text-2xl font-bold text-violet-700">{orgs.length}</span>
        <span class="text-xs text-violet-500 font-medium leading-tight">Total<br/>Orgs</span>
      </div>
    </div>
  </div>

  <div class="max-w-5xl mx-auto px-6 py-8 space-y-6">

    <!-- Feedback messages -->
    {#if error}
      <div class="flex items-center gap-3 bg-red-50 border border-red-200 rounded-xl px-4 py-3">
        <svg class="w-4 h-4 text-red-500 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <p class="text-red-700 text-sm">{error}</p>
      </div>
    {/if}
    {#if success}
      <div class="flex items-center gap-3 bg-green-50 border border-green-200 rounded-xl px-4 py-3">
        <svg class="w-4 h-4 text-green-500 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"/>
        </svg>
        <p class="text-green-700 text-sm">{success}</p>
      </div>
    {/if}

    <!-- Create org -->
    <div class="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
      <h2 class="text-sm font-semibold text-gray-700 uppercase tracking-wide mb-4">New Organization</h2>
      <form on:submit|preventDefault={createOrg} class="flex gap-3">
        <input
          bind:value={newName}
          placeholder="e.g. Acme Corp"
          class="flex-1 bg-gray-50 border border-gray-200 rounded-xl px-4 py-2.5 text-sm text-gray-900
                 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-violet-500
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
          {:else}
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
            </svg>
          {/if}
          {creating ? 'Creating…' : 'Create'}
        </button>
      </form>
    </div>

    <!-- Orgs table -->
    <div class="bg-white rounded-2xl border border-gray-200 shadow-sm overflow-hidden">
      <div class="px-6 py-4 border-b border-gray-100 flex items-center justify-between">
        <h2 class="font-semibold text-gray-900">All Organizations</h2>
        <span class="text-xs font-medium bg-gray-100 text-gray-500 px-2.5 py-1 rounded-full">{orgs.length} total</span>
      </div>

      {#if loadingOrgs}
        <div class="flex items-center justify-center gap-3 py-16 text-gray-400">
          <svg class="w-5 h-5 animate-spin" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"/>
          </svg>
          <span class="text-sm">Loading organizations…</span>
        </div>
      {:else if orgs.length === 0}
        <div class="flex flex-col items-center justify-center py-16 text-center">
          <div class="w-12 h-12 bg-gray-100 rounded-2xl flex items-center justify-center mb-3">
            <svg class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4"/>
            </svg>
          </div>
          <p class="text-sm font-medium text-gray-700">No organizations yet</p>
          <p class="text-xs text-gray-400 mt-1">Create your first organization above</p>
        </div>
      {:else}
        <table class="w-full">
          <thead>
            <tr class="bg-gray-50 border-b border-gray-100">
              <th class="text-left text-xs font-semibold text-gray-500 uppercase tracking-wide px-6 py-3">Name</th>
              <th class="text-left text-xs font-semibold text-gray-500 uppercase tracking-wide px-6 py-3">Organization ID</th>
              <th class="text-left text-xs font-semibold text-gray-500 uppercase tracking-wide px-6 py-3">Created</th>
              <th class="text-left text-xs font-semibold text-gray-500 uppercase tracking-wide px-6 py-3">Actions</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-50">
            {#each orgs as org (org._id)}
              <tr class="hover:bg-violet-50/30 transition-colors group">
                <td class="px-6 py-4">
                  {#if editingId === org._id}
                    <input
                      bind:value={editingName}
                      on:keydown={(e) => { if (e.key === 'Enter') saveEdit(org); if (e.key === 'Escape') cancelEdit(); }}
                      class="w-full border border-violet-400 rounded-lg px-3 py-1.5 text-sm
                             focus:outline-none focus:ring-2 focus:ring-violet-500 bg-white"
                      autofocus
                    />
                  {:else}
                    <div class="flex items-center gap-2.5">
                      <div class="w-7 h-7 rounded-lg bg-violet-100 flex items-center justify-center flex-shrink-0">
                        <span class="text-xs font-bold text-violet-700">{org.name[0].toUpperCase()}</span>
                      </div>
                      <span class="text-sm font-semibold text-gray-900">{org.name}</span>
                    </div>
                  {/if}
                </td>
                <td class="px-6 py-4">
                  <span class="text-xs font-mono text-gray-400 bg-gray-50 px-2 py-1 rounded-md">{org._id}</span>
                </td>
                <td class="px-6 py-4 text-sm text-gray-500">{formatDate(org.createdAt)}</td>
                <td class="px-6 py-4">
                  {#if editingId === org._id}
                    <div class="flex items-center gap-2">
                      <button
                        on:click={() => saveEdit(org)}
                        disabled={saving}
                        class="text-xs font-semibold text-white bg-violet-700 hover:bg-violet-800
                               disabled:opacity-60 rounded-lg px-3 py-1.5 transition-colors"
                      >
                        {saving ? 'Saving…' : 'Save'}
                      </button>
                      <button
                        on:click={cancelEdit}
                        class="text-xs font-medium text-gray-500 hover:text-gray-800
                               border border-gray-200 rounded-lg px-3 py-1.5 transition-colors"
                      >
                        Cancel
                      </button>
                    </div>
                  {:else}
                    <div class="flex items-center gap-2">
                      <button
                        on:click={() => startEdit(org)}
                        class="inline-flex items-center gap-1.5 text-xs font-medium text-violet-600
                               hover:text-violet-800 hover:bg-violet-100 rounded-lg px-3 py-1.5
                               transition-colors"
                      >
                        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                            d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"/>
                        </svg>
                        Edit
                      </button>
                      <button
                        on:click={() => deleteOrg(org)}
                        class="inline-flex items-center gap-1.5 text-xs font-medium text-red-500
                               hover:text-red-700 hover:bg-red-50 rounded-lg px-3 py-1.5
                               transition-colors"
                      >
                        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                            d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                        </svg>
                        Delete
                      </button>
                    </div>
                  {/if}
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      {/if}
    </div>

  </div>
</div>
