<template>

    <div>
        <form id="" v-on:submit.prevent="onSubmit">
            <div>
                <label for="cacheSize">Cache Size (KiB): </label> <input v-model="cacheSizeRef" id="cacheSize" name="cacheSize" />
            </div>
            <div>
                <label for="setAmount">Set-Anzahl: </label> <input v-model="setAmountRef" id="setAmount" name="setAmount" />
            </div>
            <div>
                <label for="addressLength">Adresslänge (bit): </label> <input v-model="addressLengthRef" id="addressLength" name="addressLength" />
            </div>
            <div>
                <label for="wordSize">Wortgröße (byte): </label> <input v-model="wordSizeRef" id="wordSize" name="wordSize" />
            </div>
            <div>
                <label for="blockSize">Blockgröße: </label> <input v-model="blockSizeRef" id="blockSize" name="blockSize" />
            </div>

            <button v-on:click="calcAddressBits">Berechen der Addressen Bits</button>
        </form>
        <div>
            <div>
                <p>Index bits: {{indexBits}}</p>
            </div>
        
            <div>
                <p>Offset bits: {{offsetBits}}</p>
            </div>

            <div>
                <p>Tag bits: {{tagBits}}</p>
            </div>
        </div>
    </div>
    <hr>


</template>

<script lang="ts">
import { defineComponent, computed, ref, watch, watchEffect } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useUrlRef } from "../url-ref";

export default defineComponent({
  
  setup() {
    const router = useRouter();
    const route = useRoute();
    const { urlRef } = useUrlRef(router, route); 
    const cacheSizeRef = urlRef("cacheSize", "");
    const setAmountRef = urlRef("setAmount", "");
    const addressLengthRef = urlRef("addressLength", "");
    const wordSizeRef = urlRef("wordSize", "");
    const blockSizeRef = urlRef("blockSize", "");

    let indexBits = -1;
    let tagBits = -1;
    let offsetBits = -1;

    const calcAddressBits = (): void => {
        const cacheSize = Number.parseInt(cacheSizeRef.value);
        const setAmount = Number.parseInt(setAmountRef.value);
        const addressLength = Number.parseInt(addressLengthRef.value);
        const wordSize = Number.parseInt(wordSizeRef.value);
        const blockSize = Number.parseInt(blockSizeRef.value);
        
        const blocksPerSet = cacheSize*1024 / (setAmount * blockSize);
        
        indexBits = Math.log2(blocksPerSet);
        offsetBits = Math.log2(blockSize);
        tagBits = addressLength - Math.log2(blocksPerSet) - Math.log2(blockSize);
    }

    return {
        cacheSizeRef,
        setAmountRef,
        addressLengthRef,
        wordSizeRef,
        blockSizeRef,
        indexBits,
        offsetBits,
        tagBits,
        calcAddressBits
    };
  },
});

</script>