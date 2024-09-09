<template>
  <main role="main" class="container">
    <div style="padding-top: 7rem" class="d-none d-lg-block"></div>
    <div class="row justify-content-md-center">
      <div class="col-lg-4">
        <div class="text-center vstack gap-3">
          <h1>Validator tool</h1>
          <div id="reader" width="400px"></div>
          <input type="text" placeholder="commitment" v-model="commitment.commitment" />
          <input type="text" placeholder="unique hash" v-model="uniqueHash.uniqueHash" />
          <button class="btn btn-info" @click="sendToBlockchain">
            Send to blockchain
          </button>
          <a href="#/" class="btn btn-primary">Back</a>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, reactive } from 'vue'
import { Html5QrcodeScanner } from "html5-qrcode";
import * as ethers from "ethers";


export default defineComponent({
  setup() {
    const commitment = ref<{ commitment?: string }>({})
    const uniqueHash = ref<{ uniqueHash?: string }>({})
    const title = ref('Validator tool');

    const init = async () => {

    }
    onMounted(() => {
      let html5QrcodeScanner = new Html5QrcodeScanner(
        "reader",
        { fps: 10, qrbox: { width: 400, height: 400 } },
        false
      );
      html5QrcodeScanner.render(
        (decodedText, decodedResult) => {
          commitment.value = decodedText;
        },
        (error) => {
          // console.log(error);
        }
      );
    });


    const sendToBlockchain = async () => {
      const abi = [
        "function registerCommitment(uint256 _uniqueHash, uint256 _commitment)",
      ];
      const provider = new ethers.providers.Web3Provider(
        (window as any).ethereum
      );
      await provider.send("eth_requestAccounts", []);
      const signer = provider.getSigner();
      const contracts = await (await fetch("contracts.json")).json();
      const contract = new ethers.Contract(contracts.zktreevote, abi, signer);
      if (!commitment.value.commitment) {
        alert("Commitment is required");
        return;
      }
      if (!uniqueHash.value.uniqueHash) {
        alert("Unique hash is required");
        return;
      }
      try {
        await contract.registerCommitment(uniqueHash.value.uniqueHash, commitment.value.commitment);
      } catch (e) {
        alert(e.reason);
      }
    }
    return {
      commitment,
      uniqueHash,
      sendToBlockchain,
    }
  }
});
</script>