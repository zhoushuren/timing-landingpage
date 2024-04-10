<script>
  export let connectedWallet = "";
  export let address = "";

  $: addressFirstPart = address.slice(0, -4);
  $: addressLastPart = address.slice(-4);

  function copyAddress() {
    navigator.clipboard.writeText(address);
    alert("Address copied to clipboard");
  }
</script>

{#if connectedWallet}
  <div class="flex flex-col gap-2">
    <div
      on:click={copyAddress}
      class="flex flex-col gap-1 p-4 rounded-xl bg-stone-50 bg-opacity-0 hover:bg-opacity-10 transition-all duration-300 ease-in-out hover:cursor-pointer"
    >
      <span>{connectedWallet}</span>
      <div class="flex text-xs text-stone-500 font-mono">
        <span class="text-nowrap overflow-hidden text-ellipsis">{addressFirstPart}</span><span>{addressLastPart}</span>
      </div>
    </div>
    <button
      class="rounded-xl text-stone-500 px-4 py-3 bg-stone-50 bg-opacity-0 hover:bg-opacity-10 font-semibold transition-all duration-300 ease-in-out"
      >Sign out</button
    >
  </div>
{/if}
