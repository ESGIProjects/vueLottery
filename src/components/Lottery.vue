<template>
  <div>
    <div class="jumbotron">
      <h1>Lottery</h1>
      <p>{{ msg }}</p>
      <p>
        <a class="btn btn-lg btn-primary" href="https://github.com/ESGIProjects/vueLottery" role="button">View on GitHub</a>
      </p>
    </div>

    <div class="container">

      
      <div class="row">

        <!-- Left side -->
        <div class="col-md-5 text-left">
          <h3>Node informations</h3>
          <p>
            The Euthereum node url to communicate with.
          </p>
            <div class="form-group">
              <label for="nodeUrl" class="col-md-3 control-label text-right">Node url</label>
              <div class="col-md-8">
                <input
                  type="text"
                  id="nodeUrl"
                  class="form-control"
                  value="nodeUrl" v-model="nodeUrl"
                   @focus="$event.target.select()">
              </div>
            </div>

          <hr />
          <h3>Access the lottery</h3>
          <p>
            Enter an existing game address or launch a new game.
          </p>

          <form class="form-horizontal">

            <div class="form-group">
              <label for="contractAddress" class="col-md-4 control-label text-right">Existing contract address</label>
              <div class="col-md-7">
                <input
                  type="text"
                  id="contractAddress"
                  class="form-control"
                  value="contractAddress" v-model="contractAddress"
                   @focus="$event.target.select()"/>
              </div>
            </div>

              <!--<div class="form-group">
                <div class="col-md-offset-4 col-md-1 text-center">
                  Or
                </div>
                <div class="col-md-2">
                  <button type="button" v-if="deploying == false"
                    class="btn btn-success"
                    @click.prevent="deployContract">Deploy a contact</button>
                    <img  v-if="deploying == true" width="32" src="@/assets/loading.gif" />
                </div>
              </div>   -->        

          </form>

          <div class="row" v-if="deployTransactionHash">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert"><strong>Txhash:</strong> {{ deployTransactionHash }}</div>
            </div>
          </div>

        <h3>Obtain the game's current gain</h3>
          <p>
            How much ETH are currently in game?
          </p>

          <form class="form-horizontal">

            <div class="form-group">
              <div class="col-md-12 text-center">
                <button type="button"
                  class="btn btn-success"
                  @click.prevent="getValue">How much?</button>
              </div>
            </div>

          </form>

          <div class="row" v-if="retreivedValue">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ retreivedValue }}</div>
            </div>
          </div>

        </div>

        <!-- Right side -->        
        <div class="col-md-6 col-md-offset-1 text-left">
                    <form class="form-horizontal">

          

            <h3>Account</h3>
            <p>
              Account addresses are used to sign the transaction.<br />
              The account must have Ether to play the game.
            </p>

            <div class="form-group">
              <label for="privateKey" class="col-md-3 control-label text-right">Private Key</label>
              <div class="col-md-7">
                <input
                  type="password"
                  id="privateKey"
                  class="form-control"
                  value="privateKey" v-model="privateKey"
                  v-show="!showPass"
                   @focus="$event.target.select()">
                 <input
                   type="text"
                   id="privateKey"
                   class="form-control"
                   value="privateKey" v-model="privateKey"
                   v-show="showPass"
                    @focus="$event.target.select()">
              </div>
              <div class="col-md-1 text-right">
                 <button type="button"
                   class="btn btn-default"
                   @click.prevent="showPass = !showPass">
                   <span class="glyphicon glyphicon-eye-open" v-show="!showPass" aria-hidden=true></span>
                   <span class="glyphicon glyphicon-eye-close" v-show="showPass" aria-hidden=true></span>
                 </button>
              </div>
            </div>


            <div class="form-group">
              <label for="address" class="col-md-3 control-label text-right">Public Key</label>
              <div class="col-md-7">
                <input
                  type="text"
                  id="address"
                  class="form-control"
                  value="address" v-model="address"
                  ref="addressField"
                  @focus="$event.target.select()" @paste="getBalance(address)">
              </div>
            </div>

            <div class="form-group">
              <label for="privateKey" class="col-md-3 text-right">Balance</label>
              <div class="col-md-8">
                {{ balance }}
              </div>
            </div>

          </form>


            <div class="form-group">
              <div class="col-md-12">
                <div class="alert alert-info" role="alert">
                  <strong>Provably fair</strong><br />
                  The lottery answer is already crypted here. After the end of the game, you will be able to decrypt it.
                  <br /> <br />
                  *hash*
                </div>
              </div>
            </div>          
        </div>


        <div class="col-md-8 col-md-offset-2 text-center">
          <h3 class="lottery">Play the lottery!</h3>
          <p>
            Click on the desired number to play the lottery! This costs X ETH. Winner gets everything!
          </p>


            <table class="lottery">
              <tr class="col-md-12">
                <td class=""><button @click.prevent="play(1)">1</button></td>
                <td class=""><button @click.prevent="play(2)">2</button></td>
                <td class=""><button @click.prevent="play(3)">3</button></td>
              </tr>
              <tr class="col-md-12">
                <td class=""><button @click.prevent="play(4)">4</button></td>
                <td class=""><button @click.prevent="play(5)">5</button></td>
                <td class=""><button @click.prevent="play(6)">6</button></td>
              </tr>
              <tr class="col-md-12">
                <td class=""><button @click.prevent="play(7)">7</button></td>
                <td class=""><button @click.prevent="play(8)">8</button></td>
                <td class=""><button @click.prevent="play(9)">9</button></td>
              </tr>
            </table>
        </div>



      </div>
    </div>
  </div>
</template>

<script>
// var solc = require('solc')
import Web3 from "web3";
import Tx from "ethereumjs-tx";
import Units from "ethereumjs-units";

let contractAbi =
  '[{"constant":true,"inputs":[],"name":"getRandom","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getFund","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getGains","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getHash","outputs":[{"name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getSize","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getSolution","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"checkWinners","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"etherreceiver","type":"address"},{"name":"amount","type":"uint256"}],"name":"fundtransfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"stop","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"number","type":"uint256"}],"name":"play","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"payMe","outputs":[{"name":"success","type":"bool"}],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]';
let contractBytecode =
  "0x6060604052341561000f57600080fd5b60d38061001d6000396000f3006060604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c14606e575b600080fd5b3415605857600080fd5b606c60048080359060200190919050506094565b005b3415607857600080fd5b607e609e565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a7230582030036eed4617b76ee4550080d2fced7bfbc3ddba9d7b7212901e539bd92c6f5a0029";

let contractSource = `pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint) {
        return storedData;
    }
}`;

let web3 = new Web3();

export default {
  name: "VueSimpleStorage",
  data() {
    return {
      title: "VueLottery",
      msg: "Lottery Ethereum smart contract interface with Vuejs",
      nodeUrl: "https://ropsten.infura.io/78ug1ovJZrudvaEsb3UV",
      contractAddress: "0x0c1a46cafd6008d8d5a3b53d8cd6ed9259941dc0",
      address: "",
      privateKey: "",
      balance: "",
      storageValue: "",
      retreivedValue: "",
      deployTransactionHash: "",
      execTransactionHash: "",
      deploying: false,
      executing: false,
      showPass: false
    };
  },
  watch: {
    address: function(val, oldVal) {
      // Get balance
      this.getBalance(val);
    },
    privateKey: function(val, oldVal) {
      // Force 0x addresses
      var regex1 = /(0x)/;
      if (regex1.test(val) == false) {
        val = "0x" + val;
      }
      // Extract account
      let account = web3.eth.accounts.privateKeyToAccount(val);
      this.address = account.address;
    }
  },
  methods: {
    // Dial node
    dialNode() {
      // Get provider
      web3.setProvider(this.nodeUrl);
    },
    // Get balance
    getBalance(address) {
      if (web3.utils.isAddress(address)) {
        // Dial node
        this.dialNode();

        // Get balance
        let self = this;
        web3.eth.getBalance(address).then(function(balance) {
          console.log("Get balance: " + balance);
          balance = Units.convert(balance, "wei", "eth");
          self.balance = balance + " Ether";
        });
      } else {
        console.log("Address not valid");
        return;
      }
    },
    createAccount() {
      console.log("Create account");

      // Dial node
      this.dialNode();

      // Create account
      let account = web3.eth.accounts.create();
      console.log(account);

      // Set account info
      this.address = account.address;
      this.privateKey = account.privateKey;

      // Get balance
      this.getBalance(account.address);
    },
    // Deploy contract
    deployContract() {
      console.log(
        "Deploy contract on node: " +
          this.nodeUrl +
          ", for address: " +
          this.address
      );

      // Dial node
      this.dialNode();

      // Get keys
      var privateKey = new Buffer(this.privateKey, "hex");
      /*
      let account = web3.eth.accounts.privateKeyToAccount(privateKey);
      console.log(account)
      */

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi);
      var contractDeploy = contract.deploy({
        data: contractBytecode
        //arguments: args
      });
      console.log("Contract: ", contract);
      // Get payload
      var payload = contractDeploy.encodeABI();

      // Save current this
      let self = this;
      self.deploying = true;
      self.contractAddress = "";

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log("Get GasPrice error: ", gasPriceError);
          self.deploying = false;
          return;
        } else {
          var gasPrice = result;
          console.log("Get GasPrice success: ", gasPrice);
          var gasPriceHex = web3.utils.numberToHex(gasPrice);

          // Get nonce
          web3.eth.getTransactionCount(self.address, function(
            nonceError,
            nonce
          ) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError);
              self.deploying = false;
              return;
            } else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce);
              console.log("Nonce hex : ", nonce);

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(300000);

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: "0x00",
                from: self.address,
                data: payload
              };
              console.log("Raw Transaction: ", rawTx);

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey);
              const tx = new Tx(rawTx);
              tx.sign(privateKey);
              const serializedTx = tx.serialize();
              console.log(
                "Raw transaction ready to be sent: ",
                "0x" + serializedTx.toString("hex")
              );

              // Send signed transaction
              web3.eth
                .sendSignedTransaction("0x" + serializedTx.toString("hex"))
                .on("error", function(error) {
                  console.log("sendRawTransaction error : ", error);
                  self.deploying = false;
                })
                .on("transactionHash", function(transactionHash) {
                  console.log("sendRawTransaction success : ", transactionHash);
                  self.deployTransactionHash = transactionHash;
                })
                .on("receipt", function(receipt) {
                  console.log(receipt.contractAddress);
                  self.contractAddress = receipt.contractAddress;
                  self.deploying = false;
                })
                .on("confirmation", function(confirmationNumber, receipt) {
                  console.log(confirmationNumber);
                  console.log(receipt);
                });
              // .then(function(newContractInstance){
              //   console.log(newContractInstance) // instance with the new contract address
              // })
            }
          });
        }
      });
    },
    // Play the game
    play(value) {
      console.log(
        "Set value:" +
          value +
          " to contract: " +
          this.contractAddress +
          " on node: " +
          this.nodeUrl
      );

      // Dial node
      this.dialNode();

      // Get keys
      var privateKey = new Buffer(this.privateKey, "hex");

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("Contract: ", contract);

      // Save current this
      let self = this;
      self.executing = true;

      // Estimate gas price
      web3.eth.getGasPrice(function(gasPriceError, result) {
        if (gasPriceError) {
          console.log("Get GasPrice error: ", gasPriceError);
          self.executing = false;
          return;
        } else {
          var gasPrice = result;
          console.log("Get GasPrice success: ", gasPrice);
          var gasPriceHex = web3.utils.numberToHex(gasPrice);

          // Get nonce
          web3.eth.getTransactionCount(self.address, function(
            nonceError,
            nonce
          ) {
            if (nonceError) {
              console.log("Nonce error : ", nonceError);
              self.executing = false;
              return;
            } else {
              // Get the current nonce
              const nonceHex = web3.utils.numberToHex(nonce);
              console.log("Nonce hex : ", nonce);

              // Execute the smart contract method
              var newValue = parseInt(value);
              var contractCallData = contract.methods.play(newValue);
              console.log("contractCallData : ", contractCallData);

              // Get payload
              var payload = contractCallData.encodeABI();

              // Set the gas limit
              const gasLimitHex = web3.utils.numberToHex(300000);

              // Create transaction
              const rawTx = {
                nonce: nonceHex,
                gasPrice: gasPriceHex,
                gasLimit: gasLimitHex,
                value: "0x2710",
                // value: web3.toWei("0.00000000000001", "ether"),
                from: self.address,
                to: self.contractAddress,
                data: payload
              };
              console.log("Raw Transaction: ", rawTx);

              // Sign and serialize the transaction
              console.log("Sign raw transaction with privatekey: ", privateKey);
              const tx = new Tx(rawTx);
              tx.sign(privateKey);
              const serializedTx = tx.serialize();
              console.log(
                "Raw transaction ready to be sent: ",
                "0x" + serializedTx.toString("hex")
              );

              

              // Send signed transaction
              web3.eth
                .sendSignedTransaction("0x" + serializedTx.toString("hex"))
                .on("error", function(error) {
                  console.log("sendRawTransaction error : ", error);
                  self.executing = false;
                })
                .on("transactionHash", function(transactionHash) {
                  console.log("sendRawTransaction success : ", transactionHash);
                  self.execTransactionHash = transactionHash;
                })
                .on("receipt", function(receipt) {
                  console.log(receipt);
                  self.executing = false;
                })
                .on("confirmation", function(confirmationNumber, receipt) {
                  console.log(confirmationNumber);
                  console.log(receipt);
                });
            }
          });
        }
      });
    },
    // Get hash value
    getHash() {
      console.log("Get contract value");

      // Dial node
      this.dialNode();

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("start call get");
      let self = this;
      contract.methods
        .getHash()
        .call()
        .then(function(result) {
          console.log("Result: " + result);
          self.retreivedValue = result;
        });
    },
    // Get gains value
    getGains() {
      console.log("Get contract value");

      // Dial node
      this.dialNode();

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("start call get");
      let self = this;
      contract.methods
        .getGains()
        .call()
        .then(function(result) {
          console.log("Result: " + result);
          self.retreivedValue = result;
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style type="text/css">
#app {
  margin-top: 0;
}

h3.lottery {
  margin-top: 20px;
}

table.lottery {
  border-collapse: collapse;
  border-width: 0;
  margin: auto;
  width: 225px;
  height: 225px;
}

table.lottery td {
  width: 75px;
  height: 75px;
  text-align: center;
}

table.lottery td button {
  width: 75px;
  height: 75px;
}
</style>
