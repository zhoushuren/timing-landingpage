<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  export let title = "Amount";
  export let value;
  export let max = Infinity;
  export let caption = "";
  export let readonly = false;
  function setMax() {
    dispatch("setMax");
  }
</script>

<div class="flex flex-col gap-2 border-stone-800 border bg-stone-900 p-4 rounded-xl">
  <span class="text-sm">{title}</span>
  <div class="flex flex-col">
    <div class="flex gap-1 items-center">
      <input
        {readonly}
        class="flex-1 h-10 border-none outline-none text-2xl bg-transparent font-bold placeholder:text-stone-600"
        class:text-stone-400={readonly}
        type="number"
        {max}
        min="0"
        placeholder="0"
        bind:value
      />
      {#if !readonly}
        <button
          on:click={setMax}
          class="rounded-full bg-stone-800 hover:bg-stone-700 transition-all duration-300 ease-in-out px-4 py-2 text-sm font-semibold"
          >Max</button
        >
      {/if}
    </div>
    {#if caption}
      <span class="opacity-70 text-xs leading-snug">{caption}</span>
      <!-- <span class="opacity-70 text-xs leading-snug">≈ ${amountValue.toLocaleString()}</span> -->
    {/if}
  </div>
</div>

<style>
  input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
</style>
