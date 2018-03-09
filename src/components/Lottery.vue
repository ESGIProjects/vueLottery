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
          <!--<h3>Node informations</h3>
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

          <hr />-->
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
                   @focus="$event.target.select()" @paste="getHash(); getState();"/>
              </div>
            </div>

          </form>

          <div v-if="provablyFairHash"> <!-- gain div -->

        <h3>Obtain the game's current gain</h3>
          <p>
            How much ETH are currently in game?
          </p>

          <form class="form-horizontal">

            <div class="form-group">
              <div class="col-md-12 text-center">
                <button type="button"
                  class="btn btn-success"
                  @click.prevent="getGains">How much?</button>
              </div>
            </div>

          </form>

          <div class="row" v-if="currentGains">
            <div class="col-md-12 text-center">
              <div class="alert alert-success" role="alert">{{ currentGains }} wei</div>
            </div>
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


            <div class="form-group" v-if="provablyFairHash">
              <div class="col-md-12">
                <div class="alert alert-info" role="alert">
                  <strong>Provably fair</strong><br />
                  The lottery answer is already crypted here. After the end of the game, you will be able to decrypt it.
                  <br /> <br />
                  {{ provablyFairHash }}
                </div>
              </div>
            </div>          
        </div>
      </div>


      <!-- bottom -->
      <div class="row" v-if="provablyFairHash && privateKey">


        <div class="col-md-5 text-center">
          <h3 class="lottery">Play the lottery!</h3>
          <p>
            Click on the desired number to play the lottery! This costs 10,000 wei. Winner gets everything!
          </p>
          <img  v-if="deploying == true" width="32" src="@/assets/loading.gif" />

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

        <div class="col-md-6 col-md-offset-1 text-left">

          <div class="form-group" v-if="gameOverTitle">
              <div class="col-md-12">
                <div class="alert alert-warning" role="alert">
                  <strong>{{ gameOverTitle }}</strong><br />
                  {{ gameOverMessage }}
                </div>
              </div>
          </div>

          <div class="form-group" v-if="gameState == true">
              <div class="col-md-12 text-center">
                <button type="button"
                  class="btn btn-success"
                  @click.prevent="getGains">Refresh</button>
              </div>
            </div>

        </div>

      </div>

      <div class="row footer"></div>



      
    </div>
  </div>
</template>

<script>
// var solc = require('solc')
import Web3 from "web3";
import Tx from "ethereumjs-tx";
import Units from "ethereumjs-units";

let contractAbi =
  '[{"constant":true,"inputs":[],"name":"getHash","outputs":[{"name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[{"name":"v","type":"uint256"}],"name":"uintToString","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"testx64","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getGains","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getNumber","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getWinners","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getPlayers","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getSolution","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getRandom","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getState","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"checkWinners","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"etherreceiver","type":"address"},{"name":"amount","type":"uint256"}],"name":"fundtransfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"number","type":"uint256"}],"name":"play","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"payMe","outputs":[{"name":"success","type":"bool"}],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]';

let web3 = new Web3();

export default {
  name: "VueLottery",
  data() {
    return {
      title: "VueLottery",
      msg: "Lottery Ethereum smart contract interface with Vuejs",
      nodeUrl: "https://ropsten.infura.io/78ug1ovJZrudvaEsb3UV",
      contractAddress: "0x19e493beeb769a23754ffb9317582a0125ce7526",
      address: "",
      privateKey: "",
      balance: "",
      provablyFairHash: "",
      currentGains: "",
      playedValue: "",
      gameState: false,
      gameOverTitle: "",
      gameOverMessage: "",
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
          web3.eth.getTransactionCount(self.address, function(nonceError, nonce) {
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
          console.log("Provably fair hash: " + result);
          self.provablyFairHash = result;
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
          console.log("Current gains: " + result);
          self.currentGains = result;
        });
    },
    // Get game state
    getState() {
      console.log("Get game state");

      // Dial node
      this.dialNode();

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("start call get");
      let self = this;
      contract.methods
        .getState()
        .call()
        .then(function(result) {
          console.log("Game state: " + result);

          self.gameState = result;
          
          if (result) {
            self.gameOverTitle = "Not over";
            self.gameOverMessage = "The game is not over! The answer and the hash method will be available soon.";
          } else {
            self.getRandom();
          }
        });
    },
    // Get random number
    getRandom() {
      console.log("Get random number");

      // Dial node
      this.dialNode();

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("start call get");
      let self = this;
      contract.methods
        .getRandom()
        .call()
        .then(function(result) {
          console.log("Random " + result);
          self.randomNumber = result;
          
          self.getSolution();
        });
    },
    // Get solution
    getSolution() {
      console.log("Get solution");

      // Dial node
      this.dialNode();

      var abi = JSON.parse(contractAbi);
      // Exec contract
      var contract = new web3.eth.Contract(abi, this.contractAddress);
      console.log("start call get");
      let self = this;
      contract.methods
        .getSolution()
        .call()
        .then(function(result) {
          console.log("Solution: " + result);
          self.gameOverTitle = "Game over!";
          self.gameOverMessage = "The game is over! The answer was " + result + ". You can verify by hashing the sum of the answer with the magic number " +self.randomNumber+" using SHA256.";
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

div.footer {
  height: 50px;
}
</style>
