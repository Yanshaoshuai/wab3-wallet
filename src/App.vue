<template>
  <!-- <Web3Api></Web3Api> -->
   <!-- <AccountSys></AccountSys> -->
    <ButtonVue></ButtonVue>
    <AccountList :walletInfo="walletInfo"></AccountList>
</template>

<script setup>
// import Web3Api from './components/test/Web3Api.vue'
// import AccountSys from './components/test/AccountSys.vue'
import ButtonVue from './components/Button.vue'
import AccountList from './components/AccountList.vue';
import store2 from 'store2'
import { onMounted, ref } from 'vue';

const walletInfo=ref([]);

onMounted(()=>{
    walletInfo.value=store2.get("walletInfo")||[];
})

</script>

<style lang="less">
body{
  padding: 10px;
}
</style>
