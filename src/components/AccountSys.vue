<template>
    <h1>助记词</h1>
    <div>{{ mnemonic }}</div>
    <h1>路径</h1>
    <div>m/44'/60/0'/0/0</div>
    <h1>地址</h1>
    <div>0x2811ad30158b1cd06b7997f24b14ac721ec38961</div>
    <h1>私钥</h1>
    <div>0x8856a222f3318829dace8057c54f5b1322ca062773c491f931bc06e56c8c8093</div>
</template>

<script setup>
import { ref } from 'vue';
import * as bip39 from 'bip39';
import ethwallet, { hdkey } from 'ethereumjs-wallet';
import Web3 from 'web3'

// const mnemonic = bip39.generateMnemonic()
const mnemonic = ref("six tail virus practice entry lonely nest panic hip ketchup hat either")
//根据助记词生成密钥对
const getByMnemonic = async () => {
    let seed = await bip39.mnemonicToSeed(mnemonic.value);
    let hdWallet = hdkey.fromMasterSeed(seed);
    let keypair = hdWallet.derivePath("m/44'/60/0'/0/0");
    //获取钱包
    let wallet = keypair.getWallet();
    //获取钱包地址
    let lowerCaseAddress = wallet.getAddressString()
    //获取钱包校验地址
    let checkAddress = wallet.getChecksumAddressString();
    //获取私钥
    let pk = wallet.getPrivateKeyString()
    //导出keystore
    //1.使用web3.js
    const web3 = new Web3(Web3.givenProvider
        || 'wss://sepolia.infura.io/ws/v3/84a3d1b0458a4a798fbb2b3177933b04')
    const keystore1 = web3.eth.accounts.encrypt(pk, '111111');
    console.log("keystore1\n", JSON.stringify(keystore1))
    //2.使用wallet对象
    const keystore2 = await wallet.toV3("111111")
    console.log("keystore2\n", JSON.stringify(keystore2))

    //通过keystore获取私钥
    //1.使用web3.js
    const pk1 = web3.eth.accounts.decrypt(keystore1, '111111').privateKey
    console.log("pk1\n", pk1)
    //2.使用wallet对象
    ethwallet.fromV3(keystore2, '111111').then((result) => {
        console.log("pk2\n", result.getPrivateKeyString())
    })

    //通过私钥获取地址
    const pkBuffer=Buffer.from(pk.slice(2),'hex')
    const walletFromPk=ethwallet.fromPrivateKey(pkBuffer)
    console.log("addressFromPk\n", walletFromPk.getAddressString())

}

getByMnemonic()
</script>

<style lang="less"></style>