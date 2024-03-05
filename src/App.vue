<template>
  <v-app>
    <v-main>
      <v-container class="fill-height">
        <v-responsive class="align-center text-center fill-height">
          <h1 class="my-5 text-white">Bridge Portal</h1>
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

          <div v-if="!walletConnected && selectedToken">
            Connect your metamask wallet to proceed...
          </div>
          <div v-if="walletConnected && selectedToken" class="align-center">
            <div class="mb-4">status: connected</div>
            <p>First, send your NFTs to the SmartBCH burn address<br/>
            <code>0x000000000000000000000000000000000000dead</code><br/>
            to burn many NFTs at once, use 'bundle transfer' on 
              <a target="_blank" :href="`https://app.withmantra.com/market/collection/${selectedToken.contract}?chain_id=10000`">Mantra</a>
            </p>

            <p className="mt-4 montserrat">Input a CashTokens receiving address from 
              <a href="https://www.paytaca.com/" target="_blank" className="underline text-blau">Paytaca</a> ,
              <a href="https://zapit.io/" target="_blank" className="underline text-blau">Zapit</a>
              or <a href="https://cashonize.com/" target="_blank" className="underline text-blau">Cashonize</a> wallet:
            </p>
            <v-text-field label="CashTokens Address" v-model="cashTokensAddr" style="width: 350px; margin: auto;"></v-text-field>
          </div>

          <div style="height: 200px;"></div>

        </v-responsive>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import {ref} from "vue"
import tokensBridge from "./assets/listTokensBridge.json"

const selectedToken = ref("")
const walletConnected= ref(false)
const cashTokensAddr = ref("")
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
