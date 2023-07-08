<template>
  <!-- <web3api /> -->
  <!-- <AccountSyx /> -->
  <div class="bg">
    <Home />
  </div>
</template>
<script setup>
import web3api from "./exercise/web3api.vue";
import AccountSyx from "./exercise/AccountSyx.vue";
import Home from "./pages/home/index.vue";
</script>
<style lang="scss">
@import "@/style/base.scss";

.bg {
  background-color: rgb(234, 234, 234);
  min-height: 100vh;
}
</style>
