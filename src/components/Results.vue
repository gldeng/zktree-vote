<template>
  <main role="main" class="container">
    <div style="padding-top: 7rem" class="d-none d-lg-block"></div>
    <div class="row justify-content-md-center">
      <div class="col-lg-4">
        <div class="text-center vstack gap-3">
          <h1>Results</h1>
          <div class="text-start">1st option: {{ options[0] }}</div>
          <div class="text-start">2nd option: {{ options[1] }}</div>
          <div class="text-start">3rd option: {{ options[2] }}</div>
          <div class="text-start">4th option: {{ options[3] }}</div>
          <a href="#/" class="btn btn-primary">Back</a>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch } from 'vue'
import * as ethers from "ethers";

export default defineComponent({
  setup() {
    const options = ref([0, 0, 0, 0]);

    const fetchResults = async () => {
      console.log('Fetching results...');
      try {
        const abi = [
          "function getOptionCounter(uint _option) external view returns (uint)",
        ];
        const provider = new ethers.providers.Web3Provider(
          (window as any).ethereum
        );
        const signer = provider.getSigner();
        const contracts = await (await fetch("contracts.json")).json();
        console.log('Contracts:', contracts);
        const contract = new ethers.Contract(contracts.zktreevote, abi, signer);
        for (let i = 0; i < 4; i++) {
          const optionCounter = await contract.getOptionCounter(i + 1);
          console.log(`Option ${i + 1} count:`, optionCounter.toString());
          options.value[i] = optionCounter.toString();
        }
        console.log('Final options:', options.value);
      } catch (error) {
        console.error('Error fetching results:', error);
      }
    }
    
    onMounted(() => {
      console.log('Component mounted');
      fetchResults();
    });

    watch(options, (newOptions) => {
      console.log('Options updated:', newOptions);
    });
    
    return {
      options,
    }
  }
})
</script>