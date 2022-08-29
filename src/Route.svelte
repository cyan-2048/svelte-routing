<script>
  import { getContext, onDestroy } from "svelte";
  import { writable } from "svelte/store";
  import { ROUTER, LOCATION } from "./contexts.js";

  export let path = "";
  export let component = null;
  export let preserve = false;

  const { registerRoute, unregisterRoute, activeRoute } = getContext(ROUTER);
  const location = getContext(LOCATION);

  const route = {
    path,
    // If no path prop is given, this Route will act as the default Route
    // that is rendered if no other Route in the Router is a match.
    default: path === "",
  };
  let routeParams = {};
  let routeProps = {};

  $: if ($activeRoute && $activeRoute.route === route) {
    routeParams = $activeRoute.params;
  }

  $: {
    const { path, component, ...rest } = $$props;
    routeProps = rest;
  }

  registerRoute(route);

  // There is no need to unregister Routes in SSR since it will all be
  // thrown away anyway.
  if (typeof window !== "undefined") {
    onDestroy(() => {
      unregisterRoute(route);
    });
  }

  let mount_writable = writable(null);

  $: visible = $activeRoute !== null && $activeRoute.route === route;
  $: if (preserve) $mount_writable = visible;
</script>

{#if preserve}
  <span style:display={visible ? null : "none"}>
    {#if component !== null}
      <svelte:component this={component} location={$location} {...routeParams} {...routeProps} />
    {:else}
      <slot writable={mount_writable} params={routeParams} location={$location} />
    {/if}
  </span>
{:else if visible}
  {#if component !== null}
    <svelte:component this={component} location={$location} {...routeParams} {...routeProps} />
  {:else}
    <slot params={routeParams} location={$location} />
  {/if}
{/if}
