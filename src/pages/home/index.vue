<template>
  <!-- <div class="nav-bar"></div> -->
  <div class="flex-center" style="min-height: 100vh; width: 100%">
    <div class="t-center" style="padding: 20px; width: 100%" v-if="!walletInfo">
      <van-button type="primary" @click="show = true" v-if="!mnemonic"
        >创建账户</van-button
      >
      <div v-if="showType == 1">
        <h2>妥善保管您的助记词</h2>
        <h3>{{ mnemonic }}</h3>
        <van-button type="primary" @click="sureSave">我已经保存</van-button>
      </div>

      <div v-if="showType == 2">
        <h2>验证助记词</h2>
        <van-cell-group inset>
          <van-field
            v-model="mnemonic2"
            rows="1"
            autosize
            type="textarea"
            placeholder="请输入助记词"
          />
        </van-cell-group>
        <van-button type="primary" @click="check" class="m-top-20"
          >确认</van-button
        >
      </div>
    </div>
    <div class="t-center" style="padding: 20px; width: 100%" v-else>
      <van-cell-group inset title="现有账户">
        <van-cell
          :title="sliceAddress(item.address)"
          v-for="(item, index) in walletInfo"
          :key="item.id"
        >
          <!-- <div slot="value">余额：{{ getMount(item.address) }}</div> -->
          <div slot="value">
            <div>余额：{{ item.balance }}ETH</div>
            <van-button
              icon="exchange"
              @click="openSendDialog(item)"
              size="mini"
            ></van-button>
          </div>
        </van-cell>
      </van-cell-group>
      <van-button type="primary" @click="addAccount" class="m-top-20"
        >添加账户</van-button
      >
    </div>

    <van-dialog
      v-model:show="show"
      title="创建钱包"
      show-cancel-button
      @confirm="addAccount"
    >
      <van-cell-group inset>
        <van-field
          v-model="password"
          type="password"
          name="密码"
          label="密码"
          placeholder="请输入密码"
          :rules="[{ required: true, message: '请输入密码' }]"
        />
      </van-cell-group>
    </van-dialog>

    <van-dialog
      v-model:show="showSend"
      title="转账"
      show-cancel-button
      @confirm="send"
    >
      <van-cell-group inset>
        <van-field
          v-model="sendInfo.toAddress"
          name="接收账户地址"
          label="接收账户地址"
          placeholder="请输入接收账户地址"
          :rules="[{ required: true, message: '请输入接收账户地址' }]"
        />
        <van-field
          v-model="sendInfo.mount"
          name="转账金额(ETH)"
          label="转账金额(ETH)"
          placeholder="请输入转账金额"
          :rules="[{ required: true, message: '请输入转账金额' }]"
        />
      </van-cell-group>
    </van-dialog>
  </div>
</template>

<script setup>
import { Buffer } from "buffer";
import { ref, reactive, computed } from "vue";
import { showNotify } from "vant";
import * as bip39 from "bip39";
import * as eWallet from "ethereumjs-wallet";
import Tx from "ethereumjs-tx";
import Web3 from "web3";
const web3 = new Web3(
  Web3.givenProvider ||
    "wss://mainnet.infura.io/ws/v3/78ba409e93434835a7e2bf1f675b1a40"
);

const show = ref(false);
const showSend = ref(false);
const sendInfo = reactive({
  fromAddress: "",
  toAddress: "",
  mount: "",
  privateKey: "",
});
const password = ref("");
const mnemonic = ref("");
const mnemonic2 = ref("");
const showType = ref(0);
const walletInfo = ref([]);
walletInfo.value = JSON.parse(localStorage.getItem("walletInfo"));
if (walletInfo.value) mnemonic.value = walletInfo.value[0].mnemonic;

const sliceAddress = (address) => {
  return address.slice(0, 8) + "..." + address.slice(address.length - 8);
};

const updateWalletInfo = () => {
  walletInfo.value.forEach(async (item, i) => {
    const balance = await web3.eth.getBalance(item.address);
    item.balance = web3.utils.fromWei(balance, "ether");
  });
};

const addAccount = () => {
  if (!walletInfo.value) {
    if (!password.value) return showNotify("请输入密码");
    mnemonic.value = bip39.generateMnemonic();
    // console.log(mnemonic.value);
    password.value = "";
    showType.value = 1;
  } else {
    createAccount();
  }
};
const sureSave = () => {
  showType.value = 2;
};
const check = async () => {
  if (mnemonic.value === mnemonic2.value) {
    createAccount();
  }
};
const createAccount = async () => {
  const seed = await bip39.mnemonicToSeed(mnemonic.value);
  const hdWallet = eWallet.hdkey.fromMasterSeed(seed);
  // console.log(walletInfo.value);
  const addressIndex = walletInfo.value ? walletInfo.value.length : 0;
  const keypair = hdWallet.derivePath(`m/44'/60'/0'/0/${addressIndex}`);
  const wallet = keypair.getWallet();
  const loweraseAddress = wallet.getAddressString();
  const checkAddress = wallet.getChecksumAddressString();
  const priKey = wallet.getPrivateKey().toString("hex");
  const keyStore = web3.eth.accounts.encrypt(priKey, password.value);
  const newWalletInfo = {
    id: addressIndex,
    address: loweraseAddress,
    privateKey: priKey,
    keyStore,
    balance: 0,
    mnemonic: mnemonic.value,
    checkAddress,
  };
  walletInfo.value = walletInfo.value
    ? [...walletInfo.value, newWalletInfo]
    : [newWalletInfo];
  localStorage.setItem("walletInfo", JSON.stringify(walletInfo.value));
  showType.value = 0;
};

const openSendDialog = (from) => {
  sendInfo.fromAddress = from.address;
  sendInfo.privateKey = from.privateKey;
  showSend.value = true;
};
const send = async () => {
  if (!sendInfo.toAddress) return showNotify("请输入接收账户地址");
  if (!sendInfo.mount) return showNotify("请输入转账金额");
  const nonce = await web3.eth.getTransactionCount(sendInfo.toAddress);
  const gasPrice = await web3.eth.getGasPrice();
  const value = web3.utils.toWei(sendInfo.mount);
  const rawTx = {
    from: sendInfo.fromAddress,
    to: sendInfo.toAddress,
    nonce,
    gasPrice,
    value,
    data: "0x0000",
  };
  console.log(rawTx);

  //转化私钥
  const privateKey = Buffer(sendInfo.privateKey, "hex");
  //gas估算
  const gas = await web3.eth.estimateGas(rawTx);
  rawTx.gas = gas;
  // console.log(rawTx);
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
    showNotify("转账成功");
    updateWalletInfo();
  });
  // trans.on("confirmation", (res) => {
  //   console.log("第n个节点确认", res);
  // });
  trans.on("error", (error) => {
    console.log(error);
    showNotify("Transaction error:" + error.message);
  });
};

updateWalletInfo();
</script>
<style scoped lang="scss">
.nav-bar {
  height: 44px;
  background-color: rgb(220, 205, 205);
}
</style>
