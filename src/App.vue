<template>
  <v-app>
    <v-main>
      <v-container class="fill-height">
        <v-responsive class="text-center fill-height">
          <v-btn style="position: absolute; right: 20px; top: 20px;">{{connectedAddress? "Connected" : "Connect"}}</v-btn>
          <h1 class="my-5 text-white" style="margin-top: 60px;">Bridge Portal</h1>
          <hr style="color: white; width: 300px; margin: auto;"><br/>

          <div className='my-4' class="swapItem">
            <a href="https://smartbch.org/" target="_blank"><img src="/images/smartbch.png" alt="smartbch" className="smartbch" style="margin-top:0px"/></a>
            ➔
            <a href="https://bitcoincash.org/" target="_blank"><img src="/images/bitcoin-cash-logo-horizontal-wt.svg" alt="BCH" className="bch" style="margin-top:0px; margin-left: 10px;"/></a>
          </div><br/>

          <label for="token">Choose which SmartBCH tokens to bridge:</label><br/>
          <v-select class="selectComponent" id="token" :items="tokensBridge" label="Token" v-model="selectedToken">
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
            <p>To bridge your {{ selectedToken.plural }}, send them to the SmartBCH burn address<br/>
            <code style="color:#222222">0x000000000000000000000000000000000000dead</code><br/>
            </p>
            <p v-if="selectedToken.hasStakeOption">If your {{ selectedToken.plural }} are staked, you need to unstake them first</p>
            <p>
            To burn many NFTs at once, use 'bundle transfer' on 
              <a target="_blank" :href="`https://app.withmantra.com/market/collection/${selectedToken.contract}?chain_id=10000`">Mantra</a>
            </p>

            <p v-if="!listToBridge.length" className="mt-7" style="color:#222222; font-weight: 700;">Listening for incoming transactions...</p>
            <div v-else>
              <p className="mt-4">Your {{ selectedToken.plural }} ready to bridge: {{ listToBridge.map(n => `#${n}`).join(", ") }}</p>
                <p className="mt-4">Input a CashTokens receiving address from 
                <a href="https://www.paytaca.com/" target="_blank" className="underline text-blau">Paytaca</a> ,
                <a href="https://zapit.io/" target="_blank" className="underline text-blau">Zapit</a>
                or <a href="https://cashonize.com/" target="_blank" className="underline text-blau">Cashonize</a> wallet:
              </p>
              <v-text-field label="CashTokens Address" v-model="cashTokensAddr" style="width: 350px; margin: auto; color: white"></v-text-field>
            </div>

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

const selectedToken = ref(undefined)
const connectedAddress= ref("")
const cashTokensAddr = ref("")
const listToBridge = ref([] as number[])

onMounted(async() => {
  try{
    // A Web3Provider wraps a standard Web3 provider, which is
    // what MetaMask injects as window.ethereum into each page
    const provider = new ethers.providers.Web3Provider(window.ethereum)

    const listAddresses = await provider.send("eth_accounts", []);
    if(listAddresses.length) connectedAddress.value = listAddresses[0];
  } catch (error){
    console.log(error)
  } 
})

watch(selectedToken, async(oldSelectedToken,newSelectedToken) => {
  // reset list to bridge when changing selected token
  listToBridge.value = []
  if(!oldSelectedToken && newSelectedToken || !connectedAddress) return
  try{
    // A Web3Provider wraps a standard Web3 provider, which is
    // what MetaMask injects as window.ethereum into each page
    const provider = new ethers.providers.Web3Provider(window.ethereum)

    const listAddresses = await provider.send("eth_requestAccounts", []);
    connectedAddress.value = listAddresses[0];
  } catch (error){
    console.log(error)
  }
})

</script>

<style scoped>
a {
  color: #0ac18e;
}
.selectComponent{
  width: 350px;
  margin:auto;
}
.tokenImage{
  width: 30px;
  margin-left: 5px;
  border-radius: 50%;
  vertical-align: center;
}
.swapItem{
  display: flex;
  justify-content: center;
  vertical-align: revert;
  align-items: center;
}
</style>
