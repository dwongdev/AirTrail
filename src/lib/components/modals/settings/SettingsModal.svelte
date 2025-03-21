<script lang="ts">
  import { cubicInOut } from 'svelte/easing';
  import { crossfade } from 'svelte/transition';

  import {
    ImportPage,
    GeneralPage,
    AppearancePage,
    UsersPage,
    ExportPage,
    OAuthPage,
    SecurityPage,
    DataPage,
  } from './pages';

  import { page } from '$app/state';
  import SettingsTabContainer from '$lib/components/modals/settings/SettingsTabContainer.svelte';
  import { Button } from '$lib/components/ui/button';
  import { Modal } from '$lib/components/ui/modal';
  import { Separator } from '$lib/components/ui/separator';
  import { cn } from '$lib/utils';
  import { isLargeScreen } from '$lib/utils/size';

  const ACCOUNT_SETTINGS = [
    { title: 'General', id: 'general' },
    { title: 'Security', id: 'security' },
    { title: 'Appearance', id: 'appearance' },
    { title: 'Import', id: 'import' },
    { title: 'Export', id: 'export' },
  ] as const;
  const ADMIN_SETTINGS = [
    { title: 'Data', id: 'data' },
    { title: 'Users', id: 'users' },
    { title: 'OAuth', id: 'oauth' },
  ] as const;
  type TabId =
    | (typeof ACCOUNT_SETTINGS)[number]['id']
    | (typeof ADMIN_SETTINGS)[number]['id'];

  let {
    open = $bindable(),
  }: {
    open: boolean;
  } = $props();

  let activeTab: TabId = $state('general');
  $effect(() => {
    if (!open) {
      activeTab = 'general';
    }
  });

  const user = $derived(page.data.user);

  const [send, receive] = crossfade({
    duration: 250,
    easing: cubicInOut,
  });
</script>

<Modal bind:open class="max-w-2xl">
  <div class="space-y-6">
    <div class="space-y-0.5">
      <h2 class="text-2xl font-bold tracking-tight">Settings</h2>
      <p class="text-muted-foreground">
        {#if !user || user.role === 'user'}
          Manage your account settings.
        {:else}
          Manage your AirTrail instance.
        {/if}
      </p>
    </div>
    <Separator class="my-6" />
    <div class="flex flex-col space-y-8 lg:flex-row lg:space-x-12 lg:space-y-0">
      <aside class="flex lg:flex-col md:-mx-4 lg:w-1/5 overflow-auto">
        <SettingsTabContainer>
          {#each ACCOUNT_SETTINGS as setting}
            {@const isActive = activeTab === setting.id}

            <Button
              onclick={() => (activeTab = setting.id)}
              variant="ghost"
              class={cn(
                !isActive && 'hover:underline',
                'relative justify-start hover:bg-transparent',
              )}
              data-sveltekit-noscroll
            >
              {#if isActive}
                <div
                  class="bg-card-hover absolute inset-0 rounded-md"
                  in:send={{ key: 'active-sidebar-tab' }}
                  out:receive={{ key: 'active-sidebar-tab' }}
                />
              {/if}
              <div class="relative">
                {setting.title}
              </div>
            </Button>
          {/each}
        </SettingsTabContainer>
        {#if user && user.role !== 'user'}
          <Separator
            class="my-2"
            orientation={$isLargeScreen ? 'horizontal' : 'vertical'}
          />
          <SettingsTabContainer>
            {#each ADMIN_SETTINGS as setting}
              {@const isActive = activeTab === setting.id}

              <Button
                onclick={() => (activeTab = setting.id)}
                variant="ghost"
                class={cn(
                  !isActive && 'hover:underline',
                  'relative justify-start hover:bg-transparent',
                )}
                data-sveltekit-noscroll
              >
                {#if isActive}
                  <div
                    class="bg-card-hover absolute inset-0 rounded-md"
                    in:send={{ key: 'active-sidebar-tab' }}
                    out:receive={{ key: 'active-sidebar-tab' }}
                  />
                {/if}
                <div class="relative">
                  {setting.title}
                </div>
              </Button>
            {/each}
          </SettingsTabContainer>
        {/if}
      </aside>
      <div class="flex-1 lg:max-w-2xl">
        {#if activeTab === 'general'}
          <GeneralPage />
        {:else if activeTab === 'security'}
          <SecurityPage />
        {:else if activeTab === 'appearance'}
          <AppearancePage />
        {:else if activeTab === 'import'}
          <ImportPage bind:open />
        {:else if activeTab === 'export'}
          <ExportPage />
        {:else if activeTab === 'data'}
          <DataPage />
        {:else if activeTab === 'users'}
          <UsersPage />
        {:else if activeTab === 'oauth'}
          <OAuthPage />
        {/if}
      </div>
    </div>
  </div>
</Modal>
