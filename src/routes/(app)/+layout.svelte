<script lang="ts">
  import { onMount } from 'svelte';
  import { Users } from '../../lib/services/Users';
  import { goto } from '$app/navigation';
  import LogoutSvg from '$lib/SVG/LogoutSvg.svelte';
  import { browser } from '$app/environment';
  import { tweened } from 'svelte/motion';
  import { quintOut } from 'svelte/easing';
  let { children } = $props();

  let isSidebarExpanded = $state(browser ? localStorage.getItem('isSidebarExpanded') === 'true' : false);
  let isSidebarHovered: boolean = $state(false);

  $effect(() => {
    // cache the sidebar state
    if (browser) localStorage.setItem('isSidebarExpanded', String(isSidebarExpanded));
  });

  let shouldShowExtendedContent = $state(false);

  $effect(() => {
    if (isSidebarHovered || isSidebarExpanded) {
      $sidebarWidth = 15;
      // set shouldShowExtendedContent to true, but delay it by 100 ms
      setTimeout(() => {
        shouldShowExtendedContent = true;
      }, 25);
    } else {
      $sidebarWidth = 5;
      shouldShowExtendedContent = false;
    }
  });

  const sidebarWidth = tweened(20, {
    duration: 200,
    easing: quintOut,
  });

  //   check if user is logged in
  onMount(async () => {
    const user = await Users.getCurrentUser();
    if (!user) {
      console.log('User not logged in');

      // redirect to login page if user is not logged in
      alert('Please login to continue');
      goto('/login');
    }
  });
</script>

<div class="drawer drawer-open">
  <input id="my-drawer" type="checkbox" class="drawer-toggle" />
  <div class="drawer-content">
    {@render children()}
  </div>
  <div class="drawer-side">
    <label for="my-drawer" aria-label="close sidebar" class="drawer-overlay"></label>
    <aside
      class={`menu p-4 min-h-full bg-base-200 text-base-content`}
      style="width: {$sidebarWidth}rem"
      onmouseenter={() => {
        // console.log('hovered');
        isSidebarHovered = true;
      }}
      onmouseleave={() => {
        // console.log('un-hovered');
        isSidebarHovered = false;
      }}
    >
      <div class="flex flex-row justify-between items-center">
        <span class="font-semibold">
          {shouldShowExtendedContent ? 'My App' : ''}
        </span>
        <button class="btn btn-ghost btn-square" onclick={() => (isSidebarExpanded = !isSidebarExpanded)}>
          {#if isSidebarExpanded}
            ←
          {:else}
            🟰
          {/if}
        </button>
      </div>
      <ul class="flex grow items-center">
        <li>
          <a href="/">
            😀
            {#if shouldShowExtendedContent}
              Sidebar Item 1
            {/if}
          </a>
        </li>
        <!-- align this to the bottom -->
      </ul>
      <li class="mt-auto">
        <button onclick={() => Users.signOut()}>
          <span class="scale-x-[-1]">
            <LogoutSvg />
          </span>
          {#if shouldShowExtendedContent}
            Log Out
          {/if}
        </button>
      </li>
    </aside>
  </div>
</div>
