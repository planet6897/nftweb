<template>
  <v-app id="inspire">
    <!-- <v-system-bar>
      <v-spacer></v-spacer>

      <v-icon>mdi-square</v-icon>

      <v-icon>mdi-circle</v-icon>

      <v-icon>mdi-triangle</v-icon>
    </v-system-bar> -->

    <v-app-bar>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>My NFT</v-toolbar-title>
      <div v-if="isLoggedIn" class="connect-button">
        <!-- <button @click="logout">Logout</button> -->
        <strong> {{ userAddress }}</strong>
      </div>
      <button v-else @click="connectAndLogin" class="connect-button">
        <img src="/metamask-icon.svg" style="width: 18px" /> Connect Wallet
      </button>
    </v-app-bar>

    <v-navigation-drawer v-model="drawer" temporary>
      <!--  -->
    </v-navigation-drawer>

    <v-main class="bg-grey-lighten-2">
      <v-container>
        <v-row v-if="contractList.length > 0">
          <template v-for="item in contractList" :key="item.address">
            <v-col class="mt-2" cols="12">
              <p><strong>Name : </strong> {{ item.tokenName }}</p>
              <p><strong>Symbol : </strong> {{ item.tokenSymbol }}</p>
              <p><strong>Address : </strong> {{ item.address }}</p>
            </v-col>

            <v-col
              v-for="inventory of filteredInventory(item.address)"
              :key="`${item.address}${inventory.name}`"
            >
              <v-sheet class="embossed-sheet" width="200" height="400">
                <v-img
                  :src="inventory.image"
                  :alt="inventory.name"
                  style="margin: 10px"
                  @click="imageClick(`${inventory.image}`)"
                ></v-img>
                <div style="border-bottom: 1px dashed grey"></div>
                <h4 class="tokenName">
                  <strong>{{ inventory.name }}</strong>
                </h4>
                <table>
                  <tr>
                    <th>trait_type</th>
                    <th>value</th>
                  </tr>
                  <template
                    v-for="attr in inventory.attributes"
                    :key="`${attr.trait_type}${attr.value}`"
                  >
                    <tr>
                      <td>{{ attr.trait_type }}</td>
                      <td>{{ attr.value }}</td>
                    </tr>
                  </template>
                </table>
              </v-sheet>
            </v-col>
          </template>
        </v-row>
        <v-row v-else>
          <h3>{{ progressMessage }}</h3>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from "vue";

const drawer = ref(null);
</script>

<script>
import axios from "axios";
import Web3 from "web3";

export default {
  name: "App",
  data() {
    return {
      drawer: null,
      isLoggedIn: false,
      // chainID: 56,
      chainID: 0,
      userAddress: null,
      contractList: [],
      tokensInfo: [],
      progressMessage: "",
    };
  },
  watch: {
    chainID() {
      this.fetchAllTokenIDs();
    },
    userAddress() {
      this.fetchAllTokenIDs();
    },
  },
  methods: {
    imageClick(imageUrl) {
      window.open(imageUrl, "_blank");
    },
    async connectAndLogin() {
      // Check if Metamask is installed
      if (typeof window.ethereum === "undefined") {
        alert("Please install Metamask to use this feature.");
        return;
      }
      try {
        // Request access to the user's accounts
        await window.ethereum.request({ method: "eth_requestAccounts" });
        // Get the current provider (usually, Metamask provider)
        const provider = new Web3(window.ethereum);
        // Get the user's account address
        const accounts = await provider.eth.getAccounts();
        this.userAddress = accounts[0];
        this.chainID = window.ethereum.networkVersion;
        this.isLoggedIn = true;

        window.ethereum.on("chainChanged", (chainId) => {
          this.chainID = parseInt(chainId);
        });

        window.ethereum.on("accountsChanged", (accounts) => {
          this.userAddress = accounts[0];
        });

        window.ethereum.on("disconnect", (error) => {
          if (error != null) {
            console.log(error);
          }

          this.contractList = [];
          this.tokensInfo = [];
        });

        console.log("Connected to Metamask with address:", this.userAddress);
      } catch (error) {
        console.error("Error connecting to Metamask:", error);
      }
    },
    async logout() {
      // Clear the user data and reset the state
      this.isLoggedIn = false;
      this.userAddress = "";

      // Reset the Web3 provider (disconnect from Metamask)
      // This step is important to remove the cached account and make sure the user re-authenticates next time
      await window.ethereum.request({ method: "eth_accounts" });
    },
    async fetchAllTokenIDs() {
      this.progressMessage = "처리중입니다.";

      if (window.ethereum.isConnected() == false) {
        return;
      }

      if (this.chainID == 0 || this.userAddress == null) {
        return;
      }

      try {
        const apiUrl = `https://appsdev.metaplanet.tech/v1/nft/list?chainID=${this.chainID}&address=${this.userAddress}`;

        const response = await axios.get(apiUrl);

        let tokensInfo = {};
        let contractList = [];

        // const apiUrl = `https://appsdev.metaplanet.tech/v1/nft/list?chainID=${this.chainID}&address=${this.userAddress}`;
        // curl "https://appsdev.metaplanet.tech/v1/nft/info?chainID=56&address=0x18817A4d5C25d3B43eE6EB1cD29304A7133fD877&
        // contractAddress=0xBf1b80294CbA70946966b645f5450a4ac4f4c19e&tokenID=0"

        if (response.data.list.length == 0) {
          this.progressMessage = "보유한 NFT가 없습니다.";
        } else {
          let nftCount = 0;
          for (let item of response.data.list) {
            nftCount += item.tokenIDs.length;
          }
          this.progressMessage = `처리중입니다... ${nftCount} 개의 NFT를 소유하고 있습니다.`;
        }

        for (let item of response.data.list) {
          let inventory = [];
          for (let tokenID of item.tokenIDs) {
            try {
              const apiUrl = `https://appsdev.metaplanet.tech/v1/nft/info?chainID=${this.chainID}&address=${this.userAddress}&contractAddress=${item.address}&tokenID=${tokenID}`;
              const response = await axios.get(apiUrl);
              inventory[tokenID] = response.data;
            } catch (err) {
              console.log(err);
            }
          }
          tokensInfo[item.address] = inventory;
          contractList.push(item);
        }
        this.tokensInfo = JSON.parse(JSON.stringify(tokensInfo));
        this.contractList = JSON.parse(JSON.stringify(contractList));
        // this.$forceUpdate(); // Manually trigger a screen update (re-render)
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    filteredInventory(address) {
      // Assuming this.tokensInfo is a data property containing the inventory data
      return this.tokensInfo[address].filter((inventory) => inventory !== null);
    },
  },
};
</script>
<style>
.embossed-sheet {
  border: 2px solid blue;
  box-shadow: 0 0 5px 2px rgba(0, 0, 0, 0.1),
    0 0 5px 4px rgba(255, 255, 255, 0.2), 0 0 5px 6px rgba(0, 0, 0, 0.1),
    0 0 5px 8px rgba(255, 255, 255, 0.2);
  background: white;
}

.connect-button {
  cursor: pointer;
  border: none;
  border-radius: 10px; /* Set the value to adjust the roundness of the border */
  background-color: rgb(143, 72, 37); /* Set the background color to red */
  color: white; /* Set the text color */
  padding: 4px 12px; /* Add padding to create some spacing around the content */
  display: flex;
  align-items: center;
  width: 158px;
  margin-right: 20px;
  white-space: nowrap; /* Prevent text from wrapping to the next line */
  overflow: clip; /* Hide overflowing content */
  text-overflow: ellipsis; /* Display ellipsis (...) for truncated text */
}

.connect-button img {
  margin-right: 8px; /* Add margin to the right of the image for spacing */
}

.tokenName {
  margin-top: 10px;
}

table {
  border-collapse: collapse;
  width: 100%;
  border: 1px solid black;
  font-size: 10px;
}

th,
td {
  border: 1px solid black;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}
</style>
