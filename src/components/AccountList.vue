<template>
    <div>
        <van-cell v-for="item in walletInfoAddressFilter" :key="item.address" :title="item.addressShow" icon="user-o">
            <template #right-icon>
                <van-button size="small" class="btn"
                 @click="getPassword(item.keystore, item.address)">{{ Number(item.balance).toFixed(6) }}</van-button>
            </template>
        </van-cell>
    </div>

    <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
        <van-cell-group>
            <van-field v-model="txPassword" label="密码" type="password" placeholder="请输入密码" />
            <van-field v-model="txToAddress" label="目标账户" type="text" placeholder="目标账户" />
            <van-field v-model="txNumber" label="金额" type="text" placeholder="转出金额" />
        </van-cell-group>
    </van-dialog>

</template>



<script setup>
import { ref, defineProps, computed } from 'vue'
import Web3 from 'web3'
import Tx from 'ethereumjs-tx'
import ethwallet, { hdkey } from 'ethereumjs-wallet';


var web3 = new Web3(Web3.givenProvider
    || 'wss://sepolia.infura.io/ws/v3/84a3d1b0458a4a798fbb2b3177933b04')


const props = defineProps({
    walletInfo: {
        type: Array,
        default: () => []
    }
})

const walletInfoAddressFilter = computed(() => {
    props.walletInfo.map(async (item) => {
        item.addressShow = item.address.slice(0, 8) + "..." + item.address.slice(-8)
        let balanceWei = await web3.eth.getBalance(item.address)
        item.balance = web3.utils.fromWei(balanceWei, 'ether')
    })
    return props.walletInfo
})

let show = ref(false)

const txPassword = ref("")
const txKeyStore = ref("")
const txAddress = ref("")
const txToAddress = ref("")
const txNumber = ref('')
const confirmPassword = async() => {
    await send()
    txPassword.value = ""
    txKeyStore.value = ""
    txAddress.value = ""
    txToAddress.value = ""
    txNumber.value = ''
}


const getPassword = (keystore, address) => {
    show.value = true
    txKeyStore.value = keystore
    txAddress.value = address
}

const send = async () => {
    let walletObj;
    try {
        walletObj = await ethwallet.fromV3(txKeyStore.value, txPassword.value)
    } catch (error) {
        alert("密码错误")
        console.log(error)
        return false;
    }
    let nonce = await web3.eth.getTransactionCount(txAddress.value)
    let gasPrice = await web3.eth.getGasPrice()

    let rawTx = {
        from: txAddress.value,
        to: txToAddress.value,
        nonce,
        gasPrice,
        value:web3.utils.toWei(txNumber.value, 'ether'),
        data: "0x0000",
    }

    //1.私钥转换
    let pk = walletObj.getPrivateKeyString()
    let pkBuffer = Buffer.from(pk.slice(2), 'hex')
    //2.gas估算
    const gas = await web3.eth.estimateGas(rawTx)
    rawTx.gas = gas;
    let tx = new Tx(rawTx)
    tx.sign(pkBuffer)
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



<style scoped>
.btn {
    width: 150px;
}
</style>