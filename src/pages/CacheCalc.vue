<template>
    <h1>Cache Calculator</h1>
    <hr>
    <h2>Adressenbits berechnen</h2>
    <cache-address-calc></cache-address-calc>
    <h2>Cache Größe berechnen</h2>
    <cache-size-calc></cache-size-calc>
    
</template>

<script lang="ts">
import CacheAddressCalc from "./../components/CacheAddressCalc.vue";
import { defineComponent, computed, ref, watch, watchEffect } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useUrlRef } from "../url-ref";
import CacheSizeCalc from "./../components/CacheSizeCalc.vue";

export default defineComponent({
  components: { CacheAddressCalc, CacheSizeCalc },
  
  setup() {


    return {
    };
  },
});

</script>