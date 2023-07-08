<template>
  <h1>助记词</h1>
  <p>{{ mnemonic }}</p>
  <h1>路径</h1>
  <p>{{ path }}</p>
  <h1>钱包地址</h1>
  <p>{{ walletInfo.address }}</p>
  <h1>钱包私钥</h1>
  <p>{{ walletInfo.privateKey }}</p>
</template>

<script setup>
import { reactive, toRefs, ref } from "vue";
import * as bip39 from "bip39";
import * as eWallet from "ethereumjs-wallet";
console.log(eWallet);
import Web3 from "web3";
import { Buffer } from "buffer";
var web3 = new Web3(
  Web3.givenProvider ||
    "wss://mainnet.infura.io/ws/v3/78ba409e93434835a7e2bf1f675b1a40"
);

//创建助记词
// const mnemonic = ref("");
// mnemonic.value = bip39.generateMnemonic();
// console.log(mnemonic.value);
const mnemonic = ref(
  "kid school drill better net tone arch among slot exclude food cliff"
);
const path = ref("m/44'/60'/0'/0/0");
const walletInfo = reactive({
  address: "",
  privateKey: "",
});

//生成密钥对 keypair
const genMnemonic = async () => {
  const seed = await bip39.mnemonicToSeed(mnemonic.value);
  // console.log(seed);
  const hdWallet = eWallet.hdkey.fromMasterSeed(seed);
  const keypair = hdWallet.derivePath(path.value);
  // console.log(keypair);

  //1.获取钱包对象
  const wallet = keypair.getWallet();
  // console.log(wallet);
  //2.获取钱包地址
  const loweraseAddress = wallet.getAddressString();
  // console.log(loweraseAddress);
  walletInfo.address = loweraseAddress;
  //3.获取钱包校验地址
  const checkAddress = wallet.getChecksumAddressString();
  //4.获取私钥
  const priKey = wallet.getPrivateKey().toString("hex");
  // console.log(priKey);
  walletInfo.privateKey = priKey;

  //私钥 → keystore ( web3js或wallet对象)
  //1.web3js
  // const keyStore1 = web3.eth.accounts.encrypt(walletInfo.privateKey, "111111");
  // console.log(JSON.stringify(keyStore1));
  //2.wallet对象
  // 从私钥创建钱包对象
  // const keyStore2 = await wallet.toV3("111111");
  // console.log(keyStore2);

  //keystore → 私钥 ( web3js或wallet对象)
  //1.web3js
  // const res1 = web3.eth.accounts.decrypt(keyStore1, "111111");
  // console.log(res1);
  //2.wallet对象
  // const wallet2 = await eWallet.fromV3(keyStore2, "111111");
  // const key = wallet2.getPrivateKey().toString("hex");
  // console.log(key);

  //通过私钥获取地址
  // const bufferPrikey = Buffer(priKey, "hex");
  // const wallet3 = eWallet.default.fromPrivateKey(bufferPrikey);
  // const loweraseAddress2 = wallet3.getAddressString();
  // console.log(bufferPrikey, wallet3, loweraseAddress2);
};
genMnemonic();
</script>
<style scoped lang="scss"></style>
