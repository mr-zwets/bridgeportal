<template>
  <v-app>
    <v-main>
      <v-container class="fill-height">
        <v-responsive class="align-center text-center fill-height">
          <h1 class="my-5">Bridge Portal</h1>

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

          <div style="height: 200px;"></div>

        </v-responsive>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import {ref} from "vue"
const selectedToken = ref("")
const tokensBridge = [
  {
    name: 'Gambling Apes Club',
    image: "https://musing-johnson-a3bf7c.netlify.app/output/gac/images/5338.png"
  },
  {
    name: 'Spice NFT',
    image: "https://images.spicenft.art/previews/2"
  },
  {
    name: 'Punk apes',
    image: "https://ipfs.punkapes.cash/ipfs/QmRgXJq2B81Goo4QyAJcVnQc4Ks9XY299B4FwRLTm5GCEb/90.png"
  }
]
</script>

<style scoped>
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
</style>
