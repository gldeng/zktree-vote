<template>
  <component :is="getMainComponent()"></component>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, onUnmounted } from 'vue'
import Home from "./Home.vue";
import ValidatorTool from "./ValidatorTool.vue";
import VoterRegistration from "./VoterRegistration.vue";
import Vote from "./Vote.vue";
import Results from "./Results.vue";

export default defineComponent({
  setup() {
    const locationHash = ref("");

    const updateHash = () => {
      locationHash.value = window.location.hash;
    };

    onMounted(() => {
      updateHash();
      window.addEventListener("hashchange", updateHash);
    });

    onUnmounted(() => {
      window.removeEventListener("hashchange", updateHash);
    });

    const getMainComponent = () => {
      const currentPath = locationHash.value.slice(1) || "/";
      if (currentPath == "/") return Home;
      if (currentPath == "/validator") return ValidatorTool;
      if (currentPath == "/registration") return VoterRegistration;
      if (currentPath == "/vote") return Vote;
      if (currentPath == "/results") return Results;
    };

    return {
      getMainComponent
    };
  }
});
</script>