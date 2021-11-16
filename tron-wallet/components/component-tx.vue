<template>
	
  <div>
    <input type="radio" id="trx" value="trx" v-model="ccx">
    <label for="one">Send TRX</label>
    <input type="radio" id="erc20" value="erc20" v-model="ccx">
    <label for="two">Send TRC20</label>

    <p>Amount:
      <input v-model="amount" placeholder="integer" @input="validate_amount">
    </p>
    <p>To:
      <input v-model="to" placeholder="TPVuBRCc3MTfzU7XC8D1n5KgT56WoD4RNe" size="50" @input="check_to_address">
      {{to_validate_err}}
    </p>
    <div v-if="ccx=='erc20'">Contract Address:
      <input v-model="contract_addr" @input="check_contract_addr" placeholder="TUzmadBzWpVdXJ8auSPy5CMSToaunSu1zu" size="50">
      {{contract_validate_err}}
      <p v-if="erc20_balance >= 0"> TRC20 Balance: {{erc20_balance}} </p>
      </div>
    <button v-on:click="submitTx">Submit</button> {{result}}
  </div>

</template>

<script>
  module.exports = {
    props: ['prvkey', 'address'],
    data() {
      return {
        ccx: 'trx',
        amount: null,
        to: null,
        contract_addr: null,
        contract: null,
        contract_validate_err: null,
        to_validate_err: null,
        result: null,
        erc20_balance: null,
      }
    }, 
    methods: {
      validate_amount() {
        return !isNaN(parseInt(this.amount));
      },
      async check_to_address() {
        try {
          tronWeb.address.toHex(this.to);
          this.to_validate_err = null;
        } catch (error) {
          console.log(error);
          this.to_validate_err = error.toString();
        }
      },
      async check_contract_addr() {
        try {
          tronWeb.address.toHex(this.contract_addr);
          this.contract = await tronWeb.contract().at(this.contract_addr);
          if (this.address) {
            var bal = await this.contract.balanceOf(this.address).call();
            this.erc20_balance = bal.toString();
          } 
          this.contract_validate_err = null;

        } catch (error) {
          console.log(error);
          this.contract_validate_err = error.toString();
        }
      },
      async submitTx() {
        if (!this.prvkey || !this.address) {
          this.result = "Please enter a private key";
          return;
        }
        try {
          tronWeb.setPrivateKey(this.prvkey);
          if (this.ccx === 'trx') {
            var tx = await tronWeb.transactionBuilder.sendTrx(
              this.to,
              parseInt(this.amount),
              this.address
            );
            const signedTx = await tronWeb.trx.sign(tx, this.prvkey);
            const receipt = await tronWeb.trx.sendRawTransaction(signedTx);
            console.log(receipt);
            this.result = "TXID: " + receipt.txid;
          } else if (this.ccx === 'erc20') {
            if (!this.contract) {
              this.result = "Please enter contract address";
              return;
            }
            let result = await this.contract.transfer(
              this.to,
              this.amount
            ).send({
              feeLimit: 10000000
            }).then(output => {
              console.log(output)
              this.result = "TXID: " + output;
            });
            console.log(result);
          }
        } catch (error) {
          console.log(error);
          this.result = error;
        }
      },
    },
    watch: {
      address: async function(val) {
        if (this.contract) {
          var bal = await this.contract.balanceOf(this.address).call();
          this.erc20_balance = bal.toString();
        }
      }
    }
  }
</script>
<style scoped>

</style>
