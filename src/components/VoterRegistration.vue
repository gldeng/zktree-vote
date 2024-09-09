<template>
  <main role="main" class="container">
    <div style="padding-top: 7rem" class="d-none d-lg-block"></div>
    <div class="row justify-content-md-center">
      <div class="col-lg-4">
        <div class="text-center vstack gap-3">
          <h1>Your commitment</h1>
          <img :src="qrcodeDataUrl" />
          <div
            v-if="commitment.commitment"
            v-html="splitTwoLines(commitment.commitment)"
          ></div>
          <button class="btn btn-info" @click="copyToClipboard">
            Copy to clipboard
          </button>
          <button class="btn btn-danger" @click="resetCommitment">
            Reset commitment
          </button>
          <a href="#/" class="btn btn-primary">Back</a>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, reactive } from 'vue'
import { generateCommitment } from "zk-merkle-tree";
import QRCode from "qrcode";
import ctc from "copy-to-clipboard";

export default defineComponent({
  setup() {
    const commitment = ref<{ commitment?: string }>({})
    const qrcodeDataUrl = ref<string>("")

    const init = async () => {
      const existingCommitment = localStorage.getItem("zktree-vote-commitment");
      const commitmentNotGenerated = !existingCommitment || existingCommitment === "undefined";
      
      if (commitmentNotGenerated) {
        commitment.value = await generateCommitment();
        localStorage.setItem("zktree-vote-commitment", JSON.stringify(commitment.value));
      } else {
        commitment.value = JSON.parse(existingCommitment);
      }

      if (commitment.value.commitment) {
        qrcodeDataUrl.value = await QRCode.toDataURL(commitment.value.commitment);
      }
    }

    onMounted(init)

    const splitTwoLines = (text: string) => {
      const half = Math.floor(text.length / 2);
      return text.substring(0, half) + "<br/>" + text.substring(half, text.length);
    }

    const copyToClipboard = () => {
      if (commitment.value.commitment) {
        ctc(commitment.value.commitment);
        alert("Successfully copied to the clipboard");
      }
    }

    const resetCommitment = () => {
      localStorage.removeItem("zktree-vote-commitment");
      init();
    }

    return {
      commitment,
      splitTwoLines,
      copyToClipboard,
      resetCommitment,
      qrcodeDataUrl,
    }
  }
})
</script>