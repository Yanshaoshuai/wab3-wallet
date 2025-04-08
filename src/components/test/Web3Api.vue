<template>
    <h1>web3钱包</h1>
    <van-divider></van-divider>
    <div>账户地址：{{ account.address }}</div>
    <div>账户私钥：{{ account.privateKey }}</div>
    <div>账户余额：{{ balance }}</div>
    <h1>转账操作</h1>
    <van-divider></van-divider>
    <van-button type="primary" @click="send">开始转账</van-button>

</template>

<script setup>
import Web3 from 'web3'
import { ref } from 'vue'
import Tx from 'ethereumjs-tx'
var web3 = new Web3(Web3.givenProvider
    || 'wss://sepolia.infura.io/ws/v3/84a3d1b0458a4a798fbb2b3177933b04')

//创建账户
// const account=web3.eth.accounts.create('1234567890')
// 账户地址
// console.log(account.address)
// 账户私钥
// console.log(account.privateKey)
const account = ref({
    address: "0x2C00279548bAa83c2108976BF7513c21b1806b4B",
    privateKey: "0xEDC2C4A52840FBE0D6DCD5735D5239CD5F5154BBd5B7DCD875E1299041F6BD4C"
})
let balance = ref(0)
web3.eth.getBalance(account.value.address)
    .then((result) => {
        // balance.value = result;
        //wei 转成eth
        balance.value = web3.utils.fromWei(result, 'ether')
    }).catch((error) => {
        console.error(error)
    })

const send = async () => {
    //构建交易对象
    //1.获取转账次数
    const nonce = await web3.eth.getTransactionCount(account.value.address)

    //2.获取gas价格
    const gasPrice = await web3.eth.getGasPrice()
    console.log("gasPrice=", gasPrice)
    //转账金额 wei为单位
    const value = web3.utils.toWei('0.01', 'ether')
    //3.构建交易对象
    let rawTx = {
        from: account.value.address,
        to: "0xCA7a83dFf6fAf16297Ef3632892C55299179e311",
        nonce,
        gasPrice,
        value,
        data: "0x0000",
    }
    console.log("rawTx=", rawTx)
    //生成serialize交易对象
    //1.私钥转换
    const privateKey = new Buffer(account.value.privateKey.slice(2), 'hex')
    //2.gas估算
    const gas = await web3.eth.estimateGas(rawTx)
    rawTx.gas = gas;
    let tx = new Tx(rawTx)
    tx.sign(privateKey)
    let serializedTx = `0x${tx.serialize().toString('hex')}`

    //开始转账
    const trans = web3.eth.sendSignedTransaction(serializedTx)
    // 交易成功
    trans.on("transactionHash", (txId) => {
        console.log("转账成功,txId=", txId)
        console.log(`交易明细:https://sepolia.etherscan.io/tx/${txId}`)
    })
    //有一个节点确认交易
    trans.on("receipt",(res)=>{
        console.log("第一个节点确认 ",res)
    })
    trans.on("confirmation",(confirmationNumber,receipt)=>{
        console.log("确认交易 ",confirmationNumber,receipt)
    })
}

</script>

<style lang="less"></style>