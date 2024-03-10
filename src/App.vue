<template>
  <v-app>
    <v-main>
      <v-container class="fill-height">
        <v-responsive class="text-center fill-height">
          <v-btn class="connectButton" @click="connectMetamask">
            {{ connectedAddress? "Connected" : "Connect" }}
          </v-btn>
          <h1 class="text-white" style="margin-top: 60px;">Bridge Portal</h1>
          <hr style="color: white; width: 300px; margin: auto;"><br/>

          <div className='my-4' class="swapItem">
            <a href="https://smartbch.org/" target="_blank"><img src="/images/smartbch.png" alt="smartbch" className="smartbch" style="margin-top:0px"/></a>
            ➔
            <a href="https://bitcoincash.org/" target="_blank"><img src="/images/bitcoin-cash-logo-horizontal-wt.svg" alt="BCH" className="bch" style="margin-top:0px; margin-left: 10px;"/></a>
          </div><br/>

          <label for="token">Choose which SmartBCH tokens to bridge:</label><br/>
          <v-select class="selectComponent text-black" id="token" :items="tokensBridge" label="Token" v-model="selectedToken">
            <template v-slot:selection="{ item, index }">
              {{item.raw.name}}
              <img class="tokenImage" :src="item.raw.image"/>
            </template>
            <template v-slot:item="{ props, item }">
              <v-list-item v-bind="props" :title="item.raw.name">
                    <template #title>
                        <span style="display: flex; justify-content: center;">
                          {{ item.raw.name }}
                          <img class="tokenImage" :src="item.raw.image"/>
                        </span>
                    </template>
                </v-list-item>
            </template>
          </v-select>

          <div v-if="!connectedAddress && selectedToken" style="color: #222222; font-weight: 700;">
            Connect your metamask wallet to proceed...
          </div>
          <div v-if="connectedAddress && selectedToken" class="align-center mt-3">
            <div v-if="selectedToken.specialMessage" class="my-3 text-black">
              <p>{{ selectedToken.specialMessage }}</p>
            </div>
            <p>To bridge your {{ selectedToken.plural }}, send them to the SmartBCH burn address<br/>
            <code style="color:#222222">0x000000000000000000000000000000000000dead</code><br/>
            </p>
            <p v-if="selectedToken.hasStakeOption">If your {{ selectedToken.plural }} are staked, you need to unstake them first</p>
            <p>
            To burn many NFTs at once, use 'bundle transfer' on 
              <a target="_blank" :href="`https://app.withmantra.com/market/collection/${selectedToken.contract}?chain_id=10000`">Mantra</a>
            </p>

            <div v-if="!listToBridge.length" className="mt-7" style="color:#222222; font-weight: 700;">
              <p v-if="!failedToFetch">Listening for incoming transactions...</p>
              <p v-else>Failed to connect to bridging backend</p>
            </div>
            <div v-else>
              <p className="mt-4 text-black">Your {{ selectedToken.plural }} ready to bridge: {{ listToBridge.map(n => `#${n}`).join(", ") }}</p>
                <p className="mt-4">Input a CashTokens receiving address from 
                <a href="https://www.paytaca.com/" target="_blank" className="underline text-blau">Paytaca</a> ,
                <a href="https://zapit.io/" target="_blank" className="underline text-blau">Zapit</a>
                or <a href="https://cashonize.com/" target="_blank" className="underline text-blau">Cashonize</a> wallet:
              </p>
              <v-text-field label="CashTokens Address" v-model="cashtokensAddr" class="inputAddr"></v-text-field>
            </div>
            <div v-if="cashtokensAddr && !isValidAddr" class="text-red">Invalid CashTokens address</div>
            
            <v-btn v-if="cashtokensAddr && isValidAddr" class="mt-5" size="large" @click="bridgeNfts">
              Bridge Your {{ listToBridge.length }} {{ selectedToken.plural }}
            </v-btn>

          </div>

        </v-responsive>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from "vue"
import { ethers } from "ethers";
import tokensBridge from "./assets/listTokensBridge.json"
import {decodeCashAddress} from '@bitauth/libauth'

declare global {
  interface Window{
    ethereum?:any
  }
}
interface nftInfoBackend{
  id: number;
  timeburned: string;
  txidsmartbch: string;
  sbchoriginaddress: string;
  nftnumber: number;
  timebridged?: string;
  txidbch?: string;
  destinationaddress?: string;
  signatureproof?: string;
}

let provider: ethers.providers.Web3Provider;
let refreshIntervalId: NodeJS.Timeout;
const selectedToken = ref(undefined as (typeof tokensBridge[0] | undefined))
const connectedAddress= ref("")
const connectedBackend = ref("")
const failedToFetch = ref(false)
const listToBridge = ref([] as number[])
const cashtokensAddr = ref("")
const isValidAddr = ref(false)

onMounted(async() => {
  try{
    // A Web3Provider wraps a standard Web3 provider, which is
    // what MetaMask injects as window.ethereum into each page
    provider = new ethers.providers.Web3Provider(window.ethereum)
    const listAddresses = await provider.send("eth_accounts", []);
    if(listAddresses.length) connectedAddress.value = ethers.utils.getAddress(listAddresses[0]);
  } catch (error){
    console.log(error)
  } 
})

function isTokenAddress(address:string) {
  const result = decodeCashAddress(address);
  if (typeof result === 'string') return false;
  const supportsTokens = (result.type === 'p2pkhWithTokens' || result.type === 'p2shWithTokens');
  return supportsTokens;
}

watch(cashtokensAddr, async(newCashtokensAddr) => {
  isValidAddr.value = isTokenAddress(newCashtokensAddr)
})

watch(selectedToken, async(newSelectedToken) => {
  // reset list to bridge when changing selected token
  listToBridge.value = [];
  failedToFetch.value = false;
  if(refreshIntervalId) clearInterval(refreshIntervalId);
  if(!connectedAddress.value) await connectMetamask()
  if(newSelectedToken && connectedAddress){
    connectedBackend.value = newSelectedToken.backendUrl;
    await getNftListAddress();
    refreshIntervalId = setInterval(getNftListAddress, 5000);
  }
})

async function getNftListAddress(){
  try{
    const rawResponse = await fetch(connectedBackend.value+'/address/'+connectedAddress.value)
    if(!rawResponse.ok) failedToFetch.value = true
    const infoAddress: nftInfoBackend[] = await rawResponse.json();

    const listNftItems = infoAddress.filter((item: nftInfoBackend) => !item.timebridged)
    listToBridge.value = listNftItems.map((item: nftInfoBackend) => item.nftnumber)
  } catch (error){
    failedToFetch.value = true
    console.log(error)
  }
}

async function connectMetamask(){
  if(!provider){
    alert("Need Metamask installed to connect.")
    return
  }
  try{
      const listAddresses = await provider.send("eth_requestAccounts", []);
      connectedAddress.value = ethers.utils.getAddress(listAddresses[0]);
    } catch (error){
      console.log(error)
    }  
}

async function bridgeNfts(){
  const signer = provider.getSigner()
  const signature = await signer.signMessage(cashtokensAddr.value);
  const rawResponse = await fetch(connectedBackend.value+'/signbridging', {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      signature, 
      sbchOriginAddress:connectedAddress.value,
      destinationAddress:cashtokensAddr.value
    })
  });
  const { txid } = await rawResponse.json();
  alert("bridging transaction succesfull, txid: " + txid);
  console.log("bridging transaction succesfull, txid: " + txid)
  listToBridge.value = []
  cashtokensAddr.value = ""
}

</script>
