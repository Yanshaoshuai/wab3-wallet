<template>
    <van-space>
        <van-button type="primary" @click="createWallet">
            创建钱包
        </van-button>
        <van-button type="primary">
            导入钱包
        </van-button>

        <van-dialog v-model:show="show" title="请输入密码" show-cancel-button @confirm="confirmPassword">
            <van-cell-group>
                <van-field v-model="password" label="密码" type="password" placeholder="请输入密码" />
            </van-cell-group>
        </van-dialog>
    </van-space>

    <template v-if="showMn">
        <p>
            {{ mnemonic }}
        </p>
        <van-button size="mini" @click="confirmSaveMnemonic">
            我已保存
        </van-button>

        <van-dialog v-model:show="showMnDialog" title="请输入助记词" show-cancel-button @confirm="confirmMn">
            <van-cell-group>
                <van-field v-model="newMnemonic" label="助记词" type="text" placeholder="请输入助记词" />
            </van-cell-group>
        </van-dialog>
    </template>


</template>



<script setup>
import { showNotify } from 'vant'
import { ref } from 'vue'
import "vant/es/dialog/style"
import "vant/es/notify/style"
import * as bip39 from 'bip39';
import { hdkey } from 'ethereumjs-wallet';
import store2 from 'store2'


let show = ref(false)
let password = ref("")
let mnemonic = ref("")
let newMnemonic = ref("")

let showMn = ref(false)
let showMnDialog = ref(false)

const createWallet = () => {
    show.value = true
}
const confirmPassword = () => {
    if (password.value === "") {
        showNotify({ type: 'danger', message: '密码不能为空' })
    } else {
        const walletInfo=store2.get("walletInfo")
        mnemonic.value=walletInfo?walletInfo[0].mnemonic:bip39.generateMnemonic()
        if(walletInfo){
            confirmMn()
        }else{
            showMn.value = true
        }
    }
}

const confirmSaveMnemonic = () => {
    showMnDialog.value = true
}

const confirmMn = async () => {
    if (newMnemonic.value === "") {
        showNotify({ type: 'danger', message: '助记词不能为空' })
    } else {
        const walletInfoOld=store2.get("walletInfo")||[];

        //助记词不匹配且没有钱包信息
        if (newMnemonic.value !== mnemonic.value&&walletInfoOld.length===0) {
            showNotify({ type: 'danger', message: '助记词不匹配' })
        } else{
            //助记词匹配成功 或已经有助记词
            showNotify({ type: 'success', message: '助记词匹配成功' })
            showMn.value = false
            //创建钱包
            const seed = await bip39.mnemonicToSeed(mnemonic.value)
            const hdWallet = hdkey.fromMasterSeed(seed)
            const addressIndex=walletInfoOld?walletInfoOld.length:0
            let keypair = hdWallet.derivePath(`m/44'/60/0'/0/${addressIndex}`)
            const wallet = keypair.getWallet()
            //获取钱包地址
            const lowerCaseAddress = wallet.getAddressString()
            //获取钱包校验地址
            const checkAddress = wallet.getChecksumAddressString()
            //获取私钥
            const pk = wallet.getPrivateKeyString()
            //导出keystore
            const keystore = await wallet.toV3(password.value)
            const walletInfo =
                {
                    id: addressIndex,
                    mnemonic: mnemonic.value,
                    password: password.value,
                    keystore: keystore,
                    address: lowerCaseAddress,
                    checkAddress: checkAddress,
                    privateKey: pk,
                    balance: 0
                }
            walletInfoOld.push(walletInfo)
            store2.set("walletInfo", walletInfoOld)
        }
    }
}
</script>



<style scoped>
p {
    user-select: all;
}
</style>