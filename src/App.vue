<template>
  <v-app id="inspire">
    <!-- <v-system-bar>
      <v-spacer></v-spacer>

      <v-icon>mdi-square</v-icon>

      <v-icon>mdi-circle</v-icon>

      <v-icon>mdi-triangle</v-icon>
    </v-system-bar> -->

    <v-app-bar style="background: rgb(50.62.69)">
      <!-- <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon> -->
      <v-toolbar-title>
        <div style="display: flex; align-items: center">
          <img
            src="/logo01.png"
            style="
              max-width: 120px;
              display: flex;
              align-items: center;
              justify-content: center;
              cursor: pointer;
            "
            onclick="window.open('https://www.metaplanet.tech', '_blank');"
          />
          <div
            style="
              font-family: 'Times New Roman', Times, serif;
              font-style: italic;
              font-weight: bold;
              font-size: 22px;
              margin-left: 50px;
            "
          >
            My NFT Gallery
          </div>
        </div>
      </v-toolbar-title>
      <v-select
        class="select_network"
        :items="networks"
        v-model="selectedNetwork"
        hide-details
      ></v-select>
      <v-text-field
        class="search_text"
        density="compact"
        variant="solo"
        label="Search contract"
        prepend-inner-icon="mdi-magnify"
        single-line
        hide-details
        clearable
        v-model="address"
        @click:clear="clearSearch"
      ></v-text-field>
      <div
        v-if="isLoggedIn"
        class="connect-button"
        v-on:click="this.address = this.userAddress"
      >
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

    <v-main class="bg-white-lighten-2 background">
      <v-container>
        <v-row v-if="contractList.length > 0">
          <template v-for="item in contractList" :key="item.address">
            <v-col class="mt-2" cols="12">
              <div class="contract_sheet">
                <p class="contract_info">
                  {{ item.tokenName }}
                </p>
                <p class="contract_detail">
                  <strong>Symbol : </strong>{{ item.tokenSymbol }}
                </p>
                <p class="contract_detail">
                  <strong>Address : </strong>
                  <a
                    :href="`${item.networkScanURL}/token/${item.address}`"
                    target="_blank"
                    >{{ item.address }}</a
                  >
                </p>
                <p class="contract_detail">
                  <strong>Quantity : </strong
                  >{{ filteredInventory(item.address).length }}
                </p>
              </div>
            </v-col>

            <v-col
              v-for="inventory of filteredInventory(item.address)"
              :key="`${item.address}${inventory.name}`"
            >
              <v-sheet
                class="embossed-sheet"
                width="250"
                height="550"
                style="
                  box-shadow: 4px 4px 8px rgba(0, 0, 0, 0.5);
                  border-radius: 15px;
                "
              >
                <v-img
                  class="v-img"
                  :src="
                    inventory.image.replace('ipfs://', 'https://ipfs.io/ipfs/')
                  "
                  :alt="inventory.name"
                  @click="itemClick(item.address, inventory)"
                ></v-img>
                <h4 class="name">
                  <strong>{{ inventory.name }}</strong>
                </h4>
                <table>
                  <tr>
                    <th>Trait_type</th>
                    <th>Value</th>
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
        <v-row v-else-if="progressMessage.length > 0">
          <div class="progressMessage">{{ progressMessage }}</div>
        </v-row>
        <v-row class="contract_info" v-else>
          <v-col cols="12" class="mt-8">
            <v-row>
              <div style="width: 400px">BoredApeYachtClub</div>
              <v-select
                class="select_address"
                :items="BoredApeYachtClub"
                v-model="address"
                hide-details
              ></v-select>
            </v-row>
          </v-col>
          <v-col cols="12" class="mt-8">
            <v-row>
              <div style="width: 400px">MutantApeYachtClub</div>
              <v-select
                class="select_address"
                :items="MutantApeYachtClub"
                v-model="address"
                hide-details
              ></v-select>
            </v-row>
          </v-col>
          <v-col cols="12" class="mt-8">
            <v-row>
              <div style="width: 400px">Dreadfulz</div>
              <v-select
                class="select_address"
                :items="Dreadfulz"
                v-model="address"
                hide-details
              ></v-select>
            </v-row>
          </v-col>
          <v-col cols="12" class="mt-8">
            <v-row>
              <div style="width: 400px">parallel</div>
              <v-select
                class="select_address"
                :items="parallel"
                v-model="address"
                hide-details
              ></v-select>
            </v-row>
          </v-col>
          <v-col cols="12" class="mt-8">
            <v-row>
              <div style="width: 400px">Art World</div>
              <v-select
                class="select_address"
                :items="ArtWorld"
                v-model="address"
                hide-details
              ></v-select>
            </v-row>
          </v-col>
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
      chainID: 1,
      userAddress: null,
      address: null,
      contractList: [],
      tokensInfo: [],
      progressMessage: "",
      networks: [
        "Ethereum Mainnet",
        "BNB Smart Chain",
        "Mumbai Polygon",
        "Polygon",
      ],
      selectedNetwork: "Ethereum Mainnet",
      ArtWorld: [
        "0x9F5851B8DadBC9d7BFaaA3109fd5eB4937346D11",
        "0x18817A4d5C25d3B43eE6EB1cD29304A7133fD877",
        "0x324ddfe367006D8a0ebd265201f45c930E85DDBB",
      ],
      BoredApeYachtClub: [
        "0x7eb413211a9DE1cd2FE8b8Bb6055636c43F7d206",
        "0xF02e86D9E0eFd57aD034FaF52201B79917fE0713",
        "0xf090Eb4c2B63e7B26E8Bb09e6Fc0cC3A7586263B",
        "0x720A4FaB08CB746fC90E88d1924a98104C0822Cf",
        "0x7A9fe22691c811ea339D9B73150e6911a5343DcA",
        "0xe3199072644455D19f58B1fd8106Ac80b3d2e780",
        "0xF22742F06e4F6d68A8d0B49b9F270bB56affAB38",
        "0xd66F8eAf84b11654a19126a98a3F55B960846Dd8",
        "0x1CFB8a2e4c2e849593882213b2468E369271dad2",
        "0xCA1257Ade6F4fA6c6834fdC42E030bE6C0f5A813",
      ],
      Dreadfulz: [
        "0xb9F0111abecC02d3d42541D6D2957Af2BE96DeA8",
        "0xa95b31421b35992D0Dc44cbBB30fB70212FD4580",
        "0x87919C56Ae8AF48246afD5bA0eA2eceA034aBAcA",
        "0x0ffB8c30736cb0C95F1aFa9BEe5294dBD3A0D779",
        "0xFd96F75963FBaFF02341e3cFf43f48C1f3B13343",
        "0x3d8703B683e29174263dA51E778b8e37b984aBc8",
        "0xECA1bB9c8d3Fd8b926372f42c8D4c6c3ed0669B3",
        "0xE95f23EE9cA2f73107E1F1B4650EED17EaFD1d09",
      ],
      MutantApeYachtClub: [
        "0xF22742F06e4F6d68A8d0B49b9F270bB56affAB38",
        "0xf090Eb4c2B63e7B26E8Bb09e6Fc0cC3A7586263B",
        "0x4F773f3FC89b73B34FB57EBc667a245D5e3812F6",
        "0xD4b06218C545C047ac3ACc7cE49d124C172DB409",
        "0xaA621B960F22911462550c078df678493c22b2ae",
        "0x7eb413211a9DE1cd2FE8b8Bb6055636c43F7d206",
        "0x6b4DF334368b09f87B3722449703060EEf284126",
        "0x720A4FaB08CB746fC90E88d1924a98104C0822Cf",
        "0x6D5a7597896A703Fe8c85775B23395a48f971305",
        "0x6C8Ee01F1f8B62E987b3D18F6F28b22a0Ada755f",
      ],
      parallel: [
        "0x435d821Ee5b346850545cB18443ca9808A9d47D0",
        "0x06e28C3c956D47eb163C8462F77CF52e3E00C2F8",
        "0xec3564D31873b17101b21aA46aAAddCaa7F55525",
        "0x8358305333cbB564148cf9FB44F3134f2208421a",
        "0xc1B172595ddC335A9DD80B50117e457cE124bc36",
        "0xe03e5A23C93d7bFc98755DB641c2321ea8555314",
        "0xA8e54b46ae93E14eedaE486a9EFCD4c7B5a5be20",
        "0xB1BBc29997826dfC94b144126135C0aabc8175a9",
        "0xCcF43Dcc4e52e0216E461955bd98B08DA53213eA",
      ],
    };
  },
  watch: {
    chainID() {
      this.fetchAllTokenIDs();
    },
    userAddress() {
      this.address = this.userAddress;
    },
    address() {
      this.fetchAllTokenIDs();
    },
    selectedNetwork() {
      switch (this.selectedNetwork) {
        case "Ethereum Mainnet":
          this.chainID = 1;
          break;
        case "BNB Smart Chain":
          this.chainID = 56;
          break;
        case "Mumbai Polygon":
          this.chainID = 80001;
          break;
        case "Polygon":
          this.chainID = 137;
          break;
      }
    },
  },
  methods: {
    clearSearch() {
      this.address = "";
      this.contractList = "";
      this.progressMessage = "";
    },
    itemClick(contractAddress, item) {
      switch (this.chainID) {
        case 1: {
          const url = `https://etherscan.io/nft/${contractAddress}/${item.tokenID}`;
          window.open(url, "_blank");
          break;
        }
        default: {
          const imageUrl = item.image.replace(
            "ipfs://",
            "https://ipfs.io/ipfs/"
          );
          window.open(imageUrl, "_blank");
        }
      }
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
        this.chainID = parseInt(window.ethereum.networkVersion);
        this.isLoggedIn = true;

        switch (this.chainID) {
          case 1:
            this.selectedNetwork = "Ethereum Mainnet";
            break;
          case 56:
            this.selectedNetwork = "BNB Smart Chain";
            break;
          case 80001:
            this.selectedNetwork = "Mumbai Polygon";
            break;
        }

        window.ethereum.on("chainChanged", (chainId) => {
          this.chainID = parseInt(chainId);
          switch (this.chainID) {
            case 1:
              this.selectedNetwork = "Ethereum Mainnet";
              break;
            case 56:
              this.selectedNetwork = "BNB Smart Chain";
              break;
            case 80001:
              this.selectedNetwork = "Mumbai Polygon";
              break;
            case 137:
              this.selectedNetwork = "Polygon";
              break;
          }
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
      if (window.ethereum.isConnected() == false) {
        return;
      }

      if (
        this.chainID == 0 ||
        this.address == null ||
        this.address.length == 0
      ) {
        return;
      }

      this.progressMessage = "처리 중입니다.";

      try {
        const apiUrl = `https://appsdev.metaplanet.tech/v1/nft/list?chainID=${this.chainID}&address=${this.address}`;

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
          this.progressMessage = `${nftCount}개의 NFT를 처리 중입니다...`;
        }

        for (let item of response.data.list) {
          let inventory = [];
          for (let tokenID of item.tokenIDs) {
            try {
              const apiUrl = `https://appsdev.metaplanet.tech/v1/nft/info?chainID=${this.chainID}&address=${this.address}&contractAddress=${item.address}&tokenID=${tokenID}`;
              const response = await axios.get(apiUrl);
              inventory[tokenID] = { ...response.data, tokenID: tokenID };
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
.background {
  background-image: url("/background.jpg");
  background-size: 100%;
  background-attachment: fixed;
}

.embossed-sheet {
  margin: 10px;
  border: 2px solid blue;
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
  /* width: 410px; */
  width: 170px;
  margin-right: 20px;
  white-space: nowrap; /* Prevent text from wrapping to the next line */
  overflow: clip; /* Hide overflowing content */
  text-overflow: ellipsis; /* Display ellipsis (...) for truncated text */
}

.select_network {
  max-width: 200px;
  margin-right: 30px;
}

.search_text {
  margin-right: 30px;
}

.connect-button img {
  margin-right: 8px; /* Add margin to the right of the image for spacing */
}

.contract_sheet {
  margin-top: 20px;
  /* background: lightblue; */
  color: white;
  border-radius: 15px;
  padding: 10px;
}

.contract_info {
  font-size: 40px;
  font-style: italic;
  font-weight: bold;
  padding: 10px;
  color: white;
}

.contract_detail {
  margin-left: 20px;
  font-size: 18px;
}

.v-img {
  margin: 10px;
  padding-top: 10px;
  border-radius: 20px;
}

.name {
  text-align: center;
}

table {
  border-collapse: collapse;
  width: 100%;
  font-size: 15px;
  color: grey;
  margin-top: 15px;
}

table > tr > th:nth-child(1),
table > tr > td:nth-child(1) {
  padding: 3px 10px 3px 10px;
  text-align: left;
}

table > tr > th:nth-child(2),
table > tr > td:nth-child(2) {
  padding: 3px 10px 3px 10px;
  text-align: right;
}

table > tr > th:nth-child(1),
table > tr > th:nth-child(2) {
  font-size: 17px;
}

.progressMessage {
  font-size: 30px;
  margin: 50px;
  font-weight: bold;
  color: white;
}

a {
  color: inherit; /* 기본 링크 색상을 상속받음 */
  text-decoration: none; /* 링크의 밑줄 제거 */
}

a:visited {
  color: inherit; /* 방문한 링크의 색상도 상속받음 */
}

.select_address {
  max-width: 400px;
  margin-left: 30px;
}
</style>
