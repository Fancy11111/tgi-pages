<template>

    <div>
        <form id="" v-on:submit.prevent="onSubmit">
            <div>
                <label for="cacheSize">Cache Size (KiBit): </label> <input v-model="cacheSizeRef" id="cacheSize" name="cacheSize" />
            </div>
            <div>
                <label for="setAmount">Set-Anzahl: </label> <input v-model="setAmountRef" id="setAmount" name="setAmount" />
            </div>
            <div>
                <label for="amountOfWays">Anzahl der Ways: </label> <input v-model="amountOfWaysRef" id="amountOfWays" name="amountOfWays" />
            </div>
            <div>
                <label for="blocksPerWord">Anzahl der Words pro Block: </label> <input v-model="blocksPerWordRef" id="blocksWordSet" name="blocksWordSet" />
            </div>
            <div>
                <label for="blockSize">Datenwortgröße (Bit): </label> <input v-model="blockSizeRef" id="blockSize" name="blockSize" />
            </div>

            <button v-on:click="calcMissingSize">Berechen der fehlenden Größe</button>
        </form>
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
    const cacheSizeRef = urlRef("cacheSize", "0");
    const blockSizeRef = urlRef("blockSize", "0")
    const blocksPerWordRef = urlRef("blocksPerWord", "0");
    const setAmountRef = urlRef("setAmount", "0");
    const amountOfWaysRef = urlRef("amountOfWays", "0");

    const calcMissingSize = () => {
        const cacheSize = Number.parseInt(cacheSizeRef.value);
        const blockSize = Number.parseInt(blockSizeRef.value);
        const blocksPerWord = Number.parseInt(blocksPerWordRef.value);
        const setAmount = Number.parseInt(setAmountRef.value);
        const amountOfWays = Number.parseInt(amountOfWaysRef.value);
        
        if(blocksPerWord == 0 || isNaN(blocksPerWord)) {
            blocksPerWordRef.value = (cacheSize / (blockSize * setAmount * amountOfWays * 1024)).toString();
        }
        else if(blockSize == 0 || isNaN(blockSize)) {
            blockSizeRef.value = (cacheSize / (blocksPerWord * setAmount * amountOfWays * 1024)).toString();
        }
        else if(setAmount == 0 || isNaN(setAmount)) {
            setAmountRef.value = (cacheSize / (blocksPerWord * blockSize * amountOfWays * 1024)).toString();
        }
        else if(cacheSize == 0 || isNaN(cacheSize)) {
            cacheSizeRef.value = (blockSize * blocksPerWord * setAmount * amountOfWays / 1024).toString();
        }
        else if(amountOfWays == 0 || isNaN(amountOfWays)) {
            amountOfWaysRef.value = (cacheSize / (blocksPerWord * blockSize * setAmount * 1024)).toString();
        }
    }

    return {
        cacheSizeRef,
        setAmountRef,
        blocksPerWordRef,
        blockSizeRef,
        calcMissingSize,
        amountOfWaysRef
    };
  },
});

</script>