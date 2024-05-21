<script setup lang="ts">
import { AES, RC4, Rabbit, TripleDES, enc, mode, pad } from 'crypto-js';
import { computedCatch } from '@/composable/computed/catchedComputed';
import { debug } from 'console';

const algos = { AES, TripleDES, Rabbit, RC4 };

//const aesModes = ['ECB', 'CBC', 'CTR', 'OFB', 'CFB'];
const aesModes = ['CBC'];
const cypherMode = ref<keyof typeof aesModes>('CBC');

const decryptMode = ref<keyof typeof aesModes>('CBC');

const cypherInput = ref('Lorem ipsum dolor sit amet');
const cypherAlgo = ref<keyof typeof algos>('AES');
const cypherSecret = ref('my secret key');
const cypheraesIv = ref('');
const cypherOutput = computed(() => {
  if (cypherAlgo.value === 'AES') {
    debugger;
    let a = enc.Utf8.parse(cypherInput.value);
    let b = enc.Utf8.parse(cypherSecret.value);
    let c = enc.Utf8.parse(cypheraesIv.value);
    console.log(a, b, c);
    return AES.encrypt(a, b, {
      iv: c,
      mode: mode.CBC,
      padding: pad.Pkcs7,
    }).ciphertext.toString();
  } else return algos[cypherAlgo.value].encrypt(cypherInput.value, cypherSecret.value).toString();
});

const decryptAesIv = ref('');

const decryptInput = ref('');
const decryptAlgo = ref<keyof typeof algos>('AES');
const decryptSecret = ref('my secret key');
const [decryptOutput, decryptError] = computedCatch(() => {
  console.log(cypherAlgo.value === 'AES');
  if (cypherAlgo.value === 'AES') {
    debugger;
    let a = enc.Base64.stringify(enc.Hex.parse(decryptInput.value));
    let b = enc.Utf8.parse(decryptSecret.value);
    let c = enc.Utf8.parse(decryptAesIv.value);
    console.log(a, b, c);
    return AES.decrypt(a, b, {
      iv: c,
      mode: mode.CBC,
      padding: pad.Pkcs7,
    }).toString(enc.Utf8);
  } else {
    return (
      algos[decryptAlgo.value].decrypt(decryptInput.value, decryptSecret.value).toString(enc.Utf8),
      {
        defaultValue: '',
        defaultErrorMessage: 'Unable to decrypt your text',
      }
    );
  }
});
</script>

<template>
  <c-card title="Encrypt">
    <div flex gap-3>
      <c-input-text
        v-model:value="cypherInput"
        label="Your text:"
        placeholder="The string to cypher"
        rows="4"
        multiline
        raw-text
        monospace
        autosize
        flex-1
      />
      <div flex flex-1 flex-col gap-2>
        <c-input-text v-model:value="cypherSecret" label="Your secret key:" clearable raw-text />

        <c-select
          v-model:value="cypherAlgo"
          label="Encryption algorithm:"
          :options="Object.keys(algos).map((label) => ({ label, value: label }))"
        />
        <c-select
          v-model:value="cypherMode"
          v-show="cypherAlgo==='AES'"
          label="MODE:"
          :options="aesModes.map((label) => ({ label, value: label }))"
        />
        <c-input-text v-model:value="cypheraesIv" v-show="cypherAlgo==='AES'" label="Your IV key:" clearable raw-text />
      </div>
    </div>
    <c-input-text
      label="Your text encrypted:"
      :value="cypherOutput"
      rows="3"
      placeholder="Your string hash"
      multiline
      monospace
      readonly
      autosize
      mt-5
    />
  </c-card>
  <c-card title="Decrypt">
    <div flex gap-3>
      <c-input-text
        v-model:value="decryptInput"
        label="Your encrypted text:"
        placeholder="The string to cypher"
        rows="4"
        multiline
        raw-text
        monospace
        flex-1
      />
      <div flex flex-1 flex-col gap-2>
        <c-input-text v-model:value="decryptSecret" label="Your secret key:" clearable raw-text />

        <c-select
          v-model:value="decryptAlgo"
          label="Encryption algorithm:"
          :options="Object.keys(algos).map((label) => ({ label, value: label }))"
        />
        <c-select
          v-model:value="decryptMode"
          v-show="decryptAlgo==='AES'"
          label="MODE:"
          :options="aesModes.map((label) => ({ label, value: label }))"
        />
        <c-input-text v-model:value="decryptAesIv" v-show="cypherAlgo==='AES'" label="Your IV key:" clearable raw-text />
      </div>
    </div>
    <c-alert v-if="decryptError" type="error" mt-12 title="Error while decrypting">{{ decryptError }}</c-alert>
    <c-input-text
      v-else
      label="Your decrypted text:"
      :value="decryptOutput"
      placeholder="Your string hash"
      rows="3"
      multiline
      monospace
      readonly
      mt-5
    />
  </c-card>
</template>
