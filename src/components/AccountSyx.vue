<template>
  <h1>助记词</h1>
  <p>{{ mnemonic }}</p>
</template>

<script setup>
import { reactive, toRefs, ref } from "vue";
import * as bip39 from "bip39";
import * as eWallet from "ethereumjs-wallet";
//创建助记词
// const mnemonic = ref("");
// mnemonic.value = bip39.generateMnemonic();
// console.log(mnemonic.value);
const mnemonic = ref(
  "kid school drill better net tone arch among slot exclude food cliff"
);

//生成密钥对 keypair
const genMnemonic = async () => {
  const seed = await bip39.mnemonicToSeed(mnemonic.value);
  // console.log(seed);
  const hdWallet = eWallet.hdkey.fromMasterSeed(seed);
  const keypair = hdWallet.derivePath("m/44'/60'/0'/0/0");
  // console.log(keypair);

  //获取私钥
  //1.获取钱包对象
  const wallet = keypair.getWallet();
  //2.获取钱包地址
  const loweraseAddress = wallet.getAddressString();
  //3.获取钱包校验地址
  const checkAddress = wallet.getChecksumAddressString();
  //4.获取私钥
  const priKey = wallet.getPrivateKey().toString("hex");
  console.log(priKey);
};
genMnemonic();
</script>
<style scoped lang="scss"></style>
