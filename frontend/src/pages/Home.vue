<template>
  <div>
    <div
      class="
        d-flex
        justify-content-between
        flex-wrap flex-md-nowrap
        align-items-center
        pt-3
        pb-2
        mb-3
        border-bottom
      "
    >
      <h1 class="h2">Welcome to Value Setter!</h1>
    </div>

    <p>
      This is a starter template with a dummy smart contract (Calc.sol) to
      jumpstart your project.
    </p>

    <p v-if="isUserConnected">
      <strong>Your current chain:</strong> {{ getChainName }}
    </p>

    <!-- <router-link v-if="isUserConnected" to="/set-value">
      <button type="button" class="btn btn-outline-primary btn-lg">
        Go set a new value!
      </button>
    </router-link> -->
    <button
      type="button"
      class="btn btn-outline-primary btn-lg"
      @click="mint()"
    >
      mint
    </button>
    ===================== TEST metamask =====================
    <p>getActiveAccount {{ getActiveAccount }}</p>
    <p>getActiveBalanceWei {{ getActiveBalanceWei }}</p>
    <p>getActiveBalanceEth {{ getActiveBalanceEth }}</p>
    <p>getChainId {{ getChainId }}</p>
    <p>getChainName {{ getChainName }}</p>
    <!-- <p>getWeb3Modal {{ getWeb3Modal }} </p> -->
    <p>isUserConnected {{ isUserConnected }}</p>

    ===================== TEST Contract =====================
    <p>getNum {{ getNum }}</p>
    <p>getCalcAbi {{ getCalcAbi }}</p>
    <p>getCalcAddress {{ getCalcAddress }}</p>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import * as crypto from "crypto";
import { ethers } from "ethers";
const web3 = new (require("web3"))();
import contractAddresses from "../contracts/addresses.json";
import ABI from "../contracts/Calc.json";


export default {
  name: "Home",
  computed: {
    ...mapGetters("accounts", [
      "getActiveAccount",
      "getActiveBalanceWei",
      "getActiveBalanceEth",
      "getChainId",
      "getChainName",
      "getProviderEthers",
      "getWeb3Modal",
      "isUserConnected",
    ]),

    ...mapGetters("contracts", ["getNum", "getCalcAbi", "getCalcAddress"])
  },
  methods: {
    getWLAddresses() {
      const WListed = {
        "0x7772005Ad71b1BC6c1E25B3a82A400B0a8FC7680": 2,
        "0xdF9A0573647079Fc6d083cFDF8FA3E4B03385a09": 1,
        "0xf44F695CeEac08875d0B36812d4953879a702fEf": 2,
      };

      const formattedWListed = {};

      Object.keys(WListed).map((WL) => {
        formattedWListed[WL.toLowerCase()] = WListed[WL];
      });

      return formattedWListed;
    },
    isWListed(address) {
      let WListedAddresses = this.getWLAddresses();
      console.warn("address ", address);
      console.warn("WListedAddresses ", WListedAddresses);
      return WListedAddresses[address];
    },
    async signing(address, allowedMintQty) {
      // Private key =  0xd3696c1b2857b790cec407a2216a5e447ba451b17407fb1487b22c53301b491a
      // Signer address =  0x086BD1DaeF343d7b71Cd879bD0e66868ac4b0106
      const SIGNER_ADDRESS_PRIVATE_KEY =
        "0xd3696c1b2857b790cec407a2216a5e447ba451b17407fb1487b22c53301b491a"; // SIGNER_ADDRESS Private key
      const nonce = crypto.randomBytes(9).toString("base64");
      const content = web3.utils.soliditySha3({
        type: "address",
        value: address
      }, {
        type: "uint256",
        value: allowedMintQty
      }, {
        type: "string",
        value: nonce
      })

    const { messageHash: hash, signature } = await web3.eth.accounts.sign(content, SIGNER_ADDRESS_PRIVATE_KEY);

      console.warn("hash ", hash);
      console.warn("nonce ", nonce);
      console.warn("signature ", signature);
      console.warn(
        "confirming pk - signer address ",
        web3.eth.accounts.privateKeyToAccount(SIGNER_ADDRESS_PRIVATE_KEY)
          .address
      );
      const recovered_address = web3.eth.accounts.recover(content, signature);
      console.warn("recovered_address ", recovered_address);
      console.warn("SIGNER_ADDRESS 0x086BD1DaeF343d7b71Cd879bD0e66868ac4b0106");
      console.warn(recovered_address === "0x086BD1DaeF343d7b71Cd879bD0e66868ac4b0106");

      return signature;
    },
    mint() {
      // console.warn("account ", this.getActiveAccount);
      // const allowedMintQty = this.isWListed(this.getActiveAccount);
      // console.warn("allowedMintQty ", allowedMintQty);

      // if(allowedMintQty){
      //   console.warn("YES WListed");

      //   //Signature Verification
      //   const signature = this.signing(this.getActiveAccount, allowedMintQty)
      //   // Call SC with signature, allowedMintQty, nonce, mintQty

      //   //MerkleTree

      // } else {
      //   console.warn("No NOT WListed");
      // }

      this.loadContract();
    },
    loadContract(){
       console.warn("contractAddress ", contractAddresses.VGX[this.getChainId]);
      console.warn("ABI ", ABI.abi);
      console.warn("getChainId ", this.getChainId);
      let signer = this.getProviderEthers; 
      console.warn("signer ", signer);

      this.contract = new ethers.Contract(contractAddresses.VGX[this.getChainId], ABI.abi, signer);
      console.warn("this.contract ", this.contract);
      console.warn("this.contract totalSupply ", );

      // this.contract.totalSupply().then(res => console.warn("totalSupply ", res.toNumber()));

    }
  },
};
</script>
