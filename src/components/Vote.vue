<template>
  <main role="main" class="container">
    <div style="padding-top: 7rem" class="d-none d-lg-block"></div>
    <div class="row justify-content-md-center">
      <div class="col-lg-4">
        <div class="text-center vstack gap-3">
          <h1>Vote</h1>
          Choose one option
          <div class="btn-group-vertical" role="group">
            <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio1" @click="option.option = 1" />
            <label class="btn btn-outline-dark" for="vbtn-radio1">1st option</label>
            <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio2" @click="option.option = 2" />
            <label class="btn btn-outline-dark" for="vbtn-radio2">2nd option</label>
            <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio3" @click="option.option = 3" />
            <label class="btn btn-outline-dark" for="vbtn-radio3">3rd option</label>
            <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio4" @click="option.option = 4" />
            <label class="btn btn-outline-dark" for="vbtn-radio4">4th option</label>
          </div>
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
import * as ethers from "ethers";
import { calculateMerkleRootAndZKProof } from "zk-merkle-tree";

// the default verifier is for 20 levels, for different number of levels, you need a new verifier circuit
const TREE_LEVELS = 20;

export default defineComponent({
  setup() {
    const option = ref({option: 0});

    const sendToBlockchain = async () => {
      if (!option.value.option) {
        alert("Please choose one!");
        return;
      }

      const commitment = JSON.parse(
        localStorage.getItem("zktree-vote-commitment")
      );
      if (!commitment) {
        alert("No commitment generated, please register!");
        return;
      }

      const abi = [
        "function vote(uint _option,uint256 _nullifier,uint256 _root,uint[2] memory _proof_a,uint[2][2] memory _proof_b,uint[2] memory _proof_c)",
      ];
      const provider = new ethers.providers.Web3Provider(
        (window as any).ethereum
      );
      await provider.send("eth_requestAccounts", []);
      const signer = provider.getSigner();
      const contracts = await (await fetch("contracts.json")).json();
      const contract = new ethers.Contract(contracts.zktreevote, abi, signer);
      console.log(contracts.zktreevote);
      
      const startTime = performance.now();
      const cd = await calculateMerkleRootAndZKProof(
        contracts.zktreevote,
        signer,
        TREE_LEVELS,
        commitment,
        "verifier.zkey"
      );
      const endTime = performance.now();
      const executionTime = endTime - startTime;
      console.log(`calculateMerkleRootAndZKProof execution time: ${executionTime} milliseconds`);
      console.log("vote ", {
          option: option.value.option,
          nullifierHash: cd.nullifierHash,
          root: cd.root,
          proof_a: cd.proof_a,
          proof_b: cd.proof_b,
          proof_c: cd.proof_c
      })
      // return;
      try {
        await contract.vote(
          option.value.option,
          cd.nullifierHash,
          cd.root,
          cd.proof_a,
          cd.proof_b,
          cd.proof_c
        );
      } catch (e) {
        alert(e.reason);
      }

    }
    return {
      option,
      sendToBlockchain,
    }
  }
})
</script>