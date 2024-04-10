<script>
  export let method = "From";
  export let tokenName = "TokenName";
  export let chainName = "BSC";
</script>

<div class="flex flex-col gap-2 border-stone-800 border bg-stone-900 p-4 rounded-xl w-44">
  <span class="text-sm">{method}</span>
  <div class="flex items-center gap-2">
    <div class="rounded-full w-12 h-12 border border-stone-800 bg-opacity-30 bg-stone-800"></div>
    <div class="flex flex-col gap-1">
      <span class="text-lg font-semibold leading-normal">{tokenName}</span>
      <span class="opacity-70 text-xs leading-snug">{chainName}</span>
    </div>
  </div>
</div>
