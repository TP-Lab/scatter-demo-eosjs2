<template>
  <div class="hello">
    <h1>scatter demo with eosjs2</h1>

    <p>{{currentAccount}}</p>

    <button @click="connect">connect</button>

    <button @click="login">login</button>

    <button @click="logout">logout</button>

    <br />

    <button @click="getPubkey">getPubkey</button>

    <button @click="account">account('eos')</button>

    <button @click="linkAccount">linkAccount</button>
    <br />

    <button @click="transfer()">transfer</button>

    <button @click="stake()">stake cpu</button>
    <br />
    <button @click="requestTransfer">requestTransfer</button>
    <button @click="getIdentityFromPermissions">getIdentityFromPermissions</button>

    <br />
    <button @click="authenticate()">authenticate</button>
  </div>
</template>

<script>
import ScatterJS from "@scatterjs/core";
import ScatterEOS from "@scatterjs/eosjs2";
import { JsonRpc, Api } from "eosjs";

import VConsole from "vconsole";
var vConsole = new VConsole();

const network = ScatterJS.Network.fromJson({
  blockchain: "eos",
  chainId: "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906",
  host: "nodes.get-scatter.com",
  port: 443,
  protocol: "https"
});

const rpc = new JsonRpc(network.fullhost());

const requiredFields = { accounts: [network] };

let scatter = "";
let api = "";

ScatterJS.plugins(new ScatterEOS());

export default {
  name: "HelloWorld",
  data() {
    return {
      currentAccount: "",
      currentPermission: "",
      currentPublicKey: ""
    };
  },
  created() {
    ScatterJS.scatter.connect("scatter-demo", { network }).then(connected => {
      if (!connected) {
        alert("no connect");
        return false;
      }

      alert("connected");
      scatter = ScatterJS.scatter;
    });
  },

  methods: {
    connect: function() {
      ScatterJS.scatter.connect("scatter-demo").then(connected => {
        if (!connected) {
          alert("no connect");
          return false;
        }

        alert("connected");
        // scatter = ScatterJS.scatter;
      });
    },
    authenticate: function() {
      scatter
        .authenticate("111222333444")
        .then(signedOrigin => {
          alert(signedOrigin);
        })
        .catch(failedAuthentication => {
          alert(failedAuthentication);
        });
    },
    requestTransfer: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      const tokenDetails = {
        contract: "eosiotptoken",
        symbol: "TPT",
        memo: "test donate",
        decimals: 4
      };
      scatter
        .requestTransfer(network, "itokenpocket", "0.0001", tokenDetails)
        .then(res => alert(res));
    },
    account: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      alert(JSON.stringify(scatter.account("eos")));
    },
    transfer: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      api
        .transact(
          {
            actions: [
              {
                account: "eosio.token",
                name: "transfer",
                authorization: [
                  {
                    actor: this.currentAccount,
                    permission: this.currentPermission
                  }
                ],
                data: {
                  from: this.currentAccount,
                  to: "itokenpocket",
                  quantity: "0.0001 EOS",
                  memo: "test eosjs2"
                }
              }
            ]
          },
          {
            blocksBehind: 3,
            expireSeconds: 30
          }
        )
        .then(res => {
          alert("yes" + res);
        })
        .catch(err => {
          alert("fail" + err);
          console.log("fail" + JSON.stringify(err));
        });
    },
    stake: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      api
        .transact(
          {
            actions: [
              {
                account: "eosio",
                name: "delegatebw",
                authorization: [
                  {
                    actor: this.currentAccount,
                    permission: this.currentPermission
                  }
                ],
                data: {
                  from: this.currentAccount,
                  receiver: this.currentAccount,
                  stake_cpu_quantity: "0.0001 EOS",
                  stake_net_quantity: "0.0001 EOS",
                  transfer: false
                }
              }
            ]
          },
          {
            blocksBehind: 3,
            expireSeconds: 30
          }
        )
        .then(data => {
          alert(JSON.stringify(data));
        })
        .catch(err => {
          console.log(JSON.stringify(err));
        });
    },
    logout: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      scatter.logout().then(a => alert(a));
    },
    login: function() {
      scatter
        .login()
        .then(id => {
          const account = scatter.identity.accounts.find(
            x => x.blockchain === "eos"
          );

          this.currentAccount = account.name;
          this.currentPermission = account.authority;
          this.currentPublicKey = account.publicKey;

          alert("get account:" + JSON.stringify(account));

          api = scatter.eos(network, Api, { rpc });
        })
        .catch(error => {
          alert("get identity error");
          console.error(error);
        });
    },
    getPubkey: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      scatter.getPublicKey("eos").then(publicKey => {
        alert(publicKey);
      });
    },
    linkAccount: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      let account = {
        name: this.currentAccount,
        authority: this.currentPermission,
        publicKey: this.currentPublicKey
      };
      scatter.linkAccount(account, network).then(linked => {
        alert(linked);
      });
    },
    getIdentityFromPermissions: function() {
      if (!this.currentAccount) {
        alert("login first");
        return;
      }
      scatter
        .getIdentityFromPermissions()
        .then(identity => {
          alert(JSON.stringify(identity));
        })
        .catch(err => {
          alert(err);
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

button {
  margin: 10px;
  padding: 8px 20px;
  background-color: #efefef;
  border-radius: 30px;
  outline: none;
  border: none;
}
</style>
