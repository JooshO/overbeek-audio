<script lang="ts">
  import { createEventDispatcher, onDestroy } from "svelte";

  const dispatch = createEventDispatcher();
  const close = () => dispatch("close");

  let modal;

  const handle_keydown = (e) => {
    if (e.key === "Escape") {
      close();
      return;
    }
  };
</script>

<svelte:window on:keydown={handle_keydown} />

<div class="modal-background" on:click={close} />

<div class="modal" role="dialog" aria-modal="true" bind:this={modal}>
  <slot name="header" />
  <hr />
  <slot />
  <hr />

  <!-- svelte-ignore a11y-autofocus -->
  <button autofocus on:click={close}>Close</button>
</div>

<style>
  .modal-background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.3);
  }

  .modal {
    position: absolute;
    left: 50%;
    top: 50%;
    width: calc(100vw - 4em);
    max-width: 32em;
    max-height: calc(100vh - 4em);
    overflow: auto;
    transform: translate(-50%, -50%);
    padding: 1em;
    border-radius: 0.2em;
    border-width: 0.2em;
    border-style: ridge;
    background: var(--theme-bg-color);
    border-color: white;
  }

  button {
    display: block;
    color: inherit;
    background-color: var(--theme-alt-bg);
  }

  button:hover {
    background-color: var(--button-hover);
  }
</style>
