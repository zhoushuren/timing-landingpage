<script>
  // UI
  import Token from "./Token.svelte";
  import { onMount, onDestroy } from 'svelte';
  import { writable } from 'svelte/store';
  import Amount from "./Amount.svelte";
  import IconSwitch from "./HeroiconsSolidArrowsRightLeft.svelte";
  import IconSwitchVertical from "./HeroiconsSolidArrowsUpDown.svelte";
  import Wallet from "./Wallet.svelte";
  import Web3Modal from 'web3modal';
  import { ethers } from 'ethers';

  import { createWeb3Modal, defaultWagmiConfig } from '@web3modal/wagmi'
  import { createClient } from 'viem'
  import { mainnet, arbitrum } from 'viem/chains'
  import { reconnect, readContract,  http, createConfig, getAccount, writeContract  } from '@wagmi/core'
  import XYZAbi from '../abi/XYZ.json'

  // 1. Define constants
  const projectId = 'f45966a4602c28d970dd9b945025158d'

  // 2. Create wagmiConfig
  const metadata = {
    name: 'Web3Modal',
    description: 'Web3Modal Example',
    url: 'https://web3modal.com', // origin must match your domain & subdomain
    icons: ['https://avatars.githubusercontent.com/u/37784886']
  }

  const chains = [ arbitrum]
  const config = defaultWagmiConfig({
    chains,
    projectId,
    metadata,
    // ...wagmiOptions // Optional - Override createConfig parameters
  })

  reconnect(config)
  // 3. Create modal
  createWeb3Modal({
    wagmiConfig: config,
    projectId,
    enableAnalytics: true, // Optional - defaults to your Cloud configuration
    enableOnramp: true, // Optional - false as default
  })
  const contract = '0xd413D215980e26c342Ae538b446a6afFc3CAC692'
  const contractWBTC = '0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f'
  let  account = ''
  let WBTCAllowance = 0
  let lockedXYZAmount = 0
  let unlockedXYZAmount = 0
  onMount(async () => {
    setTimeout(async ()=>{
      const account = getAccount(config)
      // console.log(account.address)

      const getPrice = await readContract(config,{
        abi: XYZAbi,
        address: contract,
        functionName: 'getPrice',
      })
      ABRate = new Number(getPrice) / 1e18

      const balanceWBTCPool = await readContract(config,{
        abi: XYZAbi,
        address: contractWBTC,
        functionName: 'balanceOf',
        args: [contract]
      })
      platformTokenABalance = new Number(balanceWBTCPool)  / 1e8
      const availableWBTCAccount = await readContract(config,{
        abi: XYZAbi,
        address: contractWBTC,
        functionName: 'balanceOf',
        args: [account.address]
      })
      console.log(availableWBTCAccount) // 用户一共多少BTC
      myTokenABalance = new Number(availableWBTCAccount ) / 1e8

      // const availableXYZAccount = await readContract(config,{
      //   abi: XYZAbi,
      //   address: contract,
      //   functionName: 'balanceOf',
      //   args: [account.address]
      // })
      // myTokenBBalance = new Number(availableXYZAccount ) / 1e18

      const _WBTCAllowance = await readContract(config,{
        abi: XYZAbi,
        address: contractWBTC,
        functionName: 'allowance',
        args: [account.address, contract]
      })
      console.log("WBTCAllowance:", _WBTCAllowance)
      WBTCAllowance = new Number(_WBTCAllowance)

      let lockTimeIndex = await readContract(config,{
        abi: XYZAbi,
        address: contract,
        functionName: 'lockTimeIndex',
        args: [account.address]
      })
      lockTimeIndex = new Number(lockTimeIndex)
      // console.log("lockTimeIndex:",lockTimeIndex)
      for(let i= 0; i< lockTimeIndex; i++) {
        const lockTime = await readContract(config,{
          abi: XYZAbi,
          address: contract,
          functionName: 'lockTime',
          args: [account.address, i]
        })
        // console.log("lockTime:",lockTime)
        if(parseInt(lockTime[1]) > Date.now() / 1000) {
          lockedXYZAmount += parseInt(lockTime[0] ) / 1e18
        }else{
          unlockedXYZAmount += parseInt(lockTime[0]) / 1e18
          myTokenBBalance = unlockedXYZAmount
        }
      }




    }, 1)
  });

  let connectedWallet = false

  // STATE
  const tokenA = "BTC";
  const tokenB = "XYZ";

  // swap amount
  let payAmount; // bind user input
  $: getAmount = !isBuy ? payAmount * ABRate : payAmount / ABRate;

  // exchange rate
  export let ABRate = 0;

  // token balance
  export let platformTokenABalance = 0;
  export let myTokenABalance = 0;
  export let myTokenBBalance = 0;

  let amount;
  let isBuy = true;

  // footer
  const footer = [
    { name: "Github", link: "https://github.com/zhoushuren/timing-landingpage" },
    // { name: "Whitepaper", link: "/" },
  ];

  // dashboard
  $: dashboard = [
    { name: `$${tokenA}/$${tokenB}`, value: ABRate.toLocaleString("en-US", { maximumFractionDigits: 9 }) },
    {
      name: `$${tokenA} in Platform`,
      value: platformTokenABalance.toLocaleString("en-US", { maximumFractionDigits: 2 }),
    },
  ];

  let locked = false
  async function swap() {
    if(locked) return
    locked = true
    try{
      console.log(isBuy, myTokenABalance * 1e8 ,myTokenBBalance * 1e18)
      if(isBuy) {
        // console.log("buy")
        const result = await writeContract(config, {
          abi: XYZAbi,
          address: contract,
          functionName: 'Buy',
          args: [
            myTokenABalance * 1e8,
          ],
        })
      }else{
        const result = await writeContract(config, {
          abi: XYZAbi,
          address: contract,
          functionName: 'Sell',
          args: [
            myTokenABalance * 1e8,
          ],
        })
      }
      locked = false
    }catch (e) {
      console.error(e)
      locked = false
    }
  }

  async function approve() {
    if(locked) return
    locked = true
    try{
      const result = await writeContract(config, {
        abi: XYZAbi,
        address: contractWBTC,
        functionName: 'approve',
        args: [
          contract,
          '0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff'
        ],
      })
      locked = false
      WBTCAllowance = 1
    }catch (e) {
      locked = false
    }
  }
</script>

<!-- <div class="flex flex-col h-full items-center"> -->
<!-- main -->
<div class="w-full h-full flex flex-col flex-1 self-center p-6 items-center">
  <!-- card -->
  <div><button>{account}</button></div>
  <div
    class="w-fit max-w-full flex flex-col items-stretch bg-stone-950 border-stone-800 border p-6 rounded-3xl gap-4 shadow-lg"
  >
    <w3m-button />
    <div class="p-4 w-full flex items-center justify-center">
      <a
        class=" self-center h-12 w-12 bg-white rounded-full text-xs flex items-center justify-center text-black font-extrabold"
        href="/ "
      >
        {tokenB}
      </a>
    </div>
    <div class="flex flex-col gap-4 md:flex-row md:items-stretch">
      {#each dashboard as { name, value }}
        <div class="flex flex-col gap-0 border-stone-800 border bg-stone-900 p-4 rounded-xl min-w-44">
          <span class="text-xs font-semibold text-stone-500">{name}</span>
          <span class="text-lg font-semibold">{value}</span>
        </div>
      {/each}
    </div>
    <div class="flex items-baseline justify-between">
      <h2 class="font-bold text-lg">Swap</h2>
      <!-- locked & unlocked -->
      <span class="flex gap-1 text-xs font-semibold">
        <span class="text-stone-500">${tokenB} Locked: </span>
        <span class="pr-2">{lockedXYZAmount}</span>
        <span class="text-stone-500">Unlocked: </span>
        <span class="pr-2">{unlockedXYZAmount}</span>
      </span>
    </div>
    <!-- swap -->
    <div class="flex flex-col items-stretch gap-2">
      <Amount
        on:setMax={() => (payAmount = isBuy ? myTokenBBalance : myTokenABalance)}
        max={isBuy ? myTokenBBalance : myTokenABalance}
        title={"You " + (isBuy ? "Pay" : "Sell") + " $" + (isBuy ? tokenA : tokenB)}
        bind:value={payAmount}
        caption={"Available: " + (isBuy ? myTokenBBalance : myTokenABalance)}
      ></Amount>
      <div class="relative h-0">
        <button
          on:click={() => (isBuy = !isBuy)}
          class="absolute top-[50%] left-[50%] translate-x-[-50%] translate-y-[-50%] rounded-full flex items-center justify-center w-10 h-10 bg-lime-500 text-lime-50 border border-lime-400 hover:bg-lime-400 font-semibold shadow-lg transition-all duration-300 ease-in-out hover:shadow-lime-500/30"
        >
          <IconSwitchVertical />
        </button>
      </div>
      <Amount readonly title={"You Get $" + (isBuy ? tokenB : tokenA)} bind:value={getAmount}></Amount>
    </div>
    {#if connectedWallet}
      <Wallet walletName="Metamask" walletAddress="wallet address"></Wallet>
    {/if}
    <button on:click={ WBTCAllowance ==  0 ? approve : swap}
      class="rounded-xl px-4 py-3 bg-lime-500 text-lime-50 border border-lime-400 hover:bg-lime-400 font-semibold transition-all duration-300 ease-in-out hover:shadow-lime-500/30 shadow-xl"
      >{  WBTCAllowance ==  0 ? "Approve": "Swap" }</button
    >
  </div>
  <footer class="flex justify-center p-6">
    {#each footer as { name, link }}
      <a
        class="hover:bg-stone-900/30 text-stone-400 hover:text-stone-100 transition-all px-4 py-2 rounded-full"
        href={link}>{name}</a
      >
    {/each}
  </footer>
</div>
