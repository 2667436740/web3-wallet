<template>
  <h1>钱包信息</h1>
  <p>地址{{ account.address }}</p>
  <p>私钥{{ account.privateKey }}</p>
  <p>余额{{ mount }}ETH</p>
  <h1>转账操作</h1>
  <button @click="send">发送</button>
</template>

<script setup>
import { reactive, toRefs, ref } from "vue";
import Web3 from "web3";
import { Buffer } from "buffer";
import Tx from "ethereumjs-tx";
var web3 = new Web3(
  Web3.givenProvider ||
    "wss://mainnet.infura.io/ws/v3/78ba409e93434835a7e2bf1f675b1a40"
);

// const account = web3.eth.accounts.create('123456')
// console.log(account);
const account = reactive({
  address: "0x14B7bcaA4832CD161a705E82EaAa57F5e30a40B0",
  privateKey:
    "0x10a8582c10bbd92f46cb2e642d506591b1d686e3e565e5962b3fc6607253aeb5",
});
const mount = ref(0);
web3.eth
  .getBalance(account.address)
  .then((balance) => (mount.value = web3.utils.fromWei(balance, "ether")));

const send = async () => {
  const nonce = await web3.eth.getTransactionCount(account.address);
  const gasPrice = await web3.eth.getGasPrice();
  const value = web3.utils.toWei("0.0001");
  const rawTx = {
    from: account.address,
    to: "0x92dB1666f2993128f41ae7c0c2A4399BAe04B435",
    nonce,
    gasPrice,
    value,
    data: "0x0000",
  };

  //转化私钥
  const privateKey = Buffer(account.privateKey.slice(2), "hex");
  //gas估算
  const gas = await web3.eth.estimateGas(rawTx);
  rawTx.gas = gas;
  console.log(rawTx);
  //私钥加密
  const tx = new Tx(rawTx);
  tx.sign(privateKey);
  const serializedTx = "0x" + tx.serialize().toString("hex");
  // console.log(serializedTx);
  //开始转账
  const trans = web3.eth.sendSignedTransaction(serializedTx);
  trans.on("transactionHash", (txid) => {
    console.log("交易id:", txid);
    console.log(`https://goerli.etherscan.io/tx/${txid}`);
  });
  trans.on("receipt", (res) => {
    console.log("第一个节点确认", res);
  });
  trans.on("confirmation", (res) => {
    console.log("第n个节点确认", res);
  });
};
</script>

<style scoped lang="scss"></style>
