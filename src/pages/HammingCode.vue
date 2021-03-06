<template>
  <h1>Hamming-Code</h1>

  <p>Gebe hier einen Hamming-Code ein:</p>
  <input v-model="codeBits" pattern="[01]+" />

  <p>Gebe hier die Datenbits für einen Hamming-Code ein:</p>
  <input v-model="dataBits" pattern="[01]+" />

  <pre>Hamming-Code: {{ hammingCode.code }} (bereits korrigiert)</pre>
  <pre>Datenbits: {{ hammingCode.data }}</pre>
  <pre>Anzahl der Codebits: {{ hammingCode.numCodeBits }}</pre>
  <pre>Anzahl der Paritybits: {{ hammingCode.numParityBits }}</pre>
  <pre>Anzahl der Datenbits: {{ hammingCode.numDataBits }}</pre>

  <label>
    Berechnungen anzeigen
    <input type="checkbox" v-model="showCalulations" />
  </label>

  <p>Formeln für die Paritybits:</p>
  <pre>{{ hammingCode.getFormattedHammingCodeParityFormulas(false) }}</pre>

  <pre v-if="hammingCode.errorBit != 0">Fehler an der Stelle: {{ hammingCode.errorBit }}</pre>
  <pre v-if="hammingCode.errorBit == 0">Kein erkennbarer Fehler</pre>

  <p v-if="showCalulations">Berechnung der Paritybits:</p>

  <pre v-if="showCalulations">{{
    hammingCode.getFormattedHammingCodeParityFormulas(true)
  }}</pre>

  <h2>Berechnungen</h2>

  <p>
    Wie viele Paritybits werden für einen Hamming-Code mit d Datenbits benötigt?
  </p>
  <input
    placeholder="Daten Bit Anzahl"
    v-model="dataBitCalculation.dataBits"
    type="number"
  />
  <pre>mindestens {{ dataBitCalculation.minParityBits }} Paritybits</pre>

  <p>
    Aus wie vielen Code- und Datenbits kann ein Hamming-Code bestehen, wenn
    dieser p Paritybits hat?
  </p>
  <input
    placeholder="Parity Bit Anzahl"
    v-model="parityBitCalculation.parityBits"
    type="number"
  />
  <pre>maximal {{ parityBitCalculation.maxCodeBits }} Code- und {{ parityBitCalculation.maxDataBits }} Datenbits  </pre>

  <p>Wie viele Paritybits brauche ich für einen Hamming-Code mit c Codebits?</p>
  <input
    placeholder="Code Bit Anzahl"
    v-model="codeBitCalculation.codeBits"
    type="number"
  />
  <pre>mindestens {{ codeBitCalculation.parityBits }} Paritybits</pre>
</template>

<script lang="ts">
import { defineComponent, computed, ref, watch, watchEffect } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useUrlRef } from "../url-ref";

export default defineComponent({
  components: {},
  setup() {
    const router = useRouter();
    const route = useRoute();
    const { urlRef } = useUrlRef(router, route);

    const codeBits = urlRef("hamming-code-bits", "");
    const dataBits = urlRef("hamming-data-bits", "");

    let hammingCode = ref(new HammingCode(""));
    if (codeBits.value != "") {
      hammingCode.value = new HammingCode(codeBits.value);
    } else {
      hammingCode.value = new HammingCode(dataBits.value)
    }

    const showCalulations = ref(false);

    watch(codeBits, (value) => {
      codeBits.value = codeBits.value.replace(/[^01]+$/, "");
      hammingCode.value = new HammingCode(codeBits.value, false);
      dataBits.value = "";
    });

    watch(dataBits, (value) => {
      dataBits.value = dataBits.value.replace(/[^01]+$/, "");
      hammingCode.value = new HammingCode(dataBits.value, true);
      codeBits.value = "";
    });

    const dataBitCalculation = ref(new DataBitCalculation(0));
    const parityBitCalculation = ref(new ParityBitCalculation(0));
    const codeBitCalculation = ref(new CodeBitCalculation(0));
    

    watch(dataBitCalculation, (value) => {
      value.recalculateMinParityBits();
    }, {deep: true});

    watch(parityBitCalculation, (value) => {
      value.calculateMaximumCodeAndDataBits()
    }, {deep: true});

    watch(codeBitCalculation, (value) => {
      value.calculateMinParityAndDataBits()
    }, {deep: true});

    return {
      codeBits,
      dataBits,
      hammingCode,
      showCalulations,
      dataBitCalculation,
      parityBitCalculation,
      codeBitCalculation,
    };
  },
});

interface ParityBit {
  codeIndex: number;
  involvedCodebits: Array<number>;
  value: number; 
};

class HammingCode {
  readonly code: string = "";
  readonly data: string = "";
  readonly numCodeBits: number = 0;
  readonly numDataBits: number = 0;
  readonly numParityBits: number = 0;
  readonly parityBits: Array<ParityBit> = [];
  readonly errorBit: number = 0;

  constructor(data: string, onlyDatabits?: boolean) {
    if (data == "") {
      return;
    }

    if (!onlyDatabits) {
      this.code = data;
    } else {
      this.data = data;
      this.code = this.fillDataBitsWithParityBits(); // set parity bits later
    }

    this.numCodeBits = this.code.length;

    const codeBitCalc = new CodeBitCalculation(this.numCodeBits);
    this.numParityBits = codeBitCalc.parityBits;
    this.numDataBits = codeBitCalc.dataBits;

    this.parityBits = this.calculateParityBits();

    if (!onlyDatabits) {
      this.errorBit = this.calculateError();
      this.code = this.correctCodeWord();
      this.data = this.getDataBitsFromCodeWord();
    } else {
      this.code = this.placeParityBitsInCodeWord();
      this.data = data;
    }
  }

  getFormattedHammingCodeParityFormulas(valueMode?: boolean): string {
    let output = "";
    for (let i = 0; i < this.parityBits.length; i++) {
      const prefix = "[p" + (i+1) + "]";

      const formula = this.parityBits[i].involvedCodebits.map(c => {
        if (valueMode) {
          return this.code[c-1]
        } else {
          return "c" + c
        }
      }).join( " ^ ");

      const value = this.parityBits[i].value;

      output += prefix + " = " + formula + (formula != "" ? " = " : "") + value + "\n";
    }
    return output;
  }

  private calculateParityBits() : Array<ParityBit> {
    const parityBits = new Array(this.numParityBits);

    for (let i = 0; i < this.numParityBits; i++) {
      const parityBit: ParityBit = {value: 0, involvedCodebits: [], codeIndex: i};

      for (let j = 1; j <= this.numCodeBits; j++) {
        if (isPowerOfTwo(j)) continue; 
        if (((1 << i) & j) != 0) {
          parityBit.value ^= binaryCharacterToNumber(this.code[j-1]); // '0' ascii is 48
          parityBit.involvedCodebits.push(j);
        }
      }
      parityBits[i] = parityBit;
    }
    return parityBits;
  }


  private fillDataBitsWithParityBits() : string {
    let code = "";
    let dataBitIndex = 0;
    let i = 1;
    while (dataBitIndex < this.data.length) {
      if ((i & (i - 1)) == 0) {
        code += "0";
      } else {
        code += this.data.charAt(dataBitIndex);
        dataBitIndex++;
      }
      i++;
    }
    return code;
  }

  private placeParityBitsInCodeWord() : string {
    let correctedCode = "";
    let parityBitIndex = 0;
    for (let i = 1; i <= this.numCodeBits; i++) {
      if (isPowerOfTwo(i)) {
        correctedCode += this.parityBits[parityBitIndex].value;
        parityBitIndex++;
      } else {
        correctedCode += this.code[i - 1];
      }
    } 
    return correctedCode;
  }

  private calculateError() : number {
    let error = 0;
    for (let i = 0; i < this.numParityBits; i++) {
      const codebitIndex = Math.floor(Math.pow(2, i));

      if (binaryCharacterToNumber(this.code[codebitIndex - 1]) != this.parityBits[i].value) {
        error += codebitIndex;
      }
    }
    return error;
  }

  private correctCodeWord() {
    return replaceCharAt(this.code, this.errorBit-1, (flipBinaryString(this.code[this.errorBit-1])));
  }

  private getDataBitsFromCodeWord() : string { 
    let dataword = "";
    for (let i = 1; i <= this.numCodeBits; i++) {
      if (isPowerOfTwo(i)) continue;
      dataword += this.code.charAt(i - 1);
    }
    return dataword;
  } 
}

class DataBitCalculation {
  dataBits: number = 0;
  
  minParityBits: number = 0;

  constructor(dataBits: number) {
    this.dataBits = dataBits;
    this.recalculateMinParityBits();
  }

  recalculateMinParityBits() {
    let parityBits = 0;
    let dataBitIndex = 0;
    let codeBitIndex = 1;
    while (dataBitIndex < this.dataBits) {
      if (isPowerOfTwo(codeBitIndex)) {
        parityBits++;
      } else {
        dataBitIndex++;
      }
      codeBitIndex++;
    }
    this.minParityBits = parityBits;
  }
}

class ParityBitCalculation {
  parityBits: number = 0;

  maxCodeBits: number = 0;
  maxDataBits: number = 0;

  constructor(parityBits: number) {
    this.parityBits = parityBits;
    this.calculateMaximumCodeAndDataBits();
  }

  calculateMaximumCodeAndDataBits() {
    this.maxCodeBits = Math.pow(2, this.parityBits) - 1;
    this.maxDataBits = this.maxCodeBits - this.parityBits;
  }
}

class CodeBitCalculation {
  codeBits: number = 0;

  parityBits: number = 0;
  dataBits: number = 0;

  constructor(codeBits: number) {
    this.codeBits = codeBits;
    this.calculateMinParityAndDataBits();
  }

  calculateMinParityAndDataBits() {
    this.parityBits = Math.max(Math.floor(Math.log2(this.codeBits)) + 1, 0);
    this.dataBits = this.codeBits - this.parityBits;
  }
}

function binaryCharacterToNumber(character: string) : number {
  if (character.length != 1) return -1;
  return character.charCodeAt(0) - 48; // '0' is ascii 48
}

function toSizedBinaryString(number: number, length: number): string {
  return number.toString(2).padStart(length, "0");
}

function isPowerOfTwo(n: number): boolean {
  return (n & (n - 1)) == 0;
}

function flipBinaryString(character: string) : string {
  if (character.length != 1) return character;
  if (character[0] == "0") {
    return "1";
  } else if (character[0] == "1") {
    return "0";
  }
  return character; 
}

function replaceCharAt(value: string, index: number, replacement: string) : string {
  if (replacement.length != 1) return value; 
  return value.substring(0, index) + replacement + value.substring(index + replacement.length);
}

</script>
