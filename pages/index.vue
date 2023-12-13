<template>
  <div
    class="mx-auto flex min-w-0 max-w-7xl flex-1 flex-col px-4 sm:px-10 text-white"
  >
    <!-- logo -->
    <div class="flex items-center justify-end py-8">
      <a href="/" class="mr-auto">
        <img src="~/assets/icons/logo.svg" class="text-white" alt="logo" />
      </a>
    </div>

    <div
      class="mx-auto mt-10 h-full w-full rounded-[30px] bg-gray-900 px-5 py-10 sm:mt-[92px] sm:w-[800px] sm:px-[50px]"
    >
      <div class="relative">
        <!-- header -->
        <div class="mb-7 flex flex-col items-center">
          <img src="~/assets/icons/logo-1.svg" alt="logo-1" class="w-12 h-12" />
          <h1 class="mt-5 text-center text-2xl font-semibold leading-10">
            {{ isConnected ? "Connected!" : "Connect to Avocado" }}
          </h1>
          <p class="text-center text-xs font-medium leading-6 text-gray-400">
            &amp; enjoy the most user-friendly web3 experience!
          </p>
        </div>

        <!-- wallet connection part -->
        <div class="flex flex-col gap-4 p-2" v-if="!isConnected">
          <Button @click="connectWallet">
            <div class="flex flex-1 items-center gap-[15px]">
              <img src="~/assets/icons/metamask.svg" alt="metamask" />
              <span class="text-[16px]">{{
                isConnecting ? "Connecting..." : "MetaMask"
              }}</span>
            </div>

            <img src="~/assets/icons/arrow-right.svg" alt="arrow-right" />
          </Button>
        </div>

        <!-- main -->
        <div class="flex flex-col gap-5" v-else>
          <div class="grid grid-cols-2 gap-4">
            <!-- EOA -->
            <div class="flex flex-col gap-2">
              <h3 class="text-center font-bold">EOA</h3>

              <div class="flex flex-col gap-4">
                <!-- USDC on Polygon -->
                <div class="flex items-center justify-between">
                  <span>USDC(Polygon):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingEOABalance" />
                    <span v-else>{{ usdcOnPolygonOfEOA }}</span>
                    <span>USDC</span>
                  </span>
                </div>

                <!-- DAI on Arbi -->
                <div class="flex items-center justify-between">
                  <span>DAI(Arbi):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingEOABalance" />
                    <span v-else>{{ daiOnArbiOfEOA }}</span>
                    <span>DAI</span>
                  </span>
                </div>

                <!-- USDT on Optimism -->
                <div class="flex items-center justify-between">
                  <span>USDT(Optimism):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingEOABalance" />
                    <span v-else>{{ usdtOnOptimismOfEOA }}</span>
                    <span>UDST</span>
                  </span>
                </div>
              </div>
            </div>

            <!-- Avocado -->
            <div class="flex flex-col gap-2">
              <h3 class="text-center font-bold uppercase">Avocado</h3>

              <div class="flex flex-col gap-4">
                <!-- USDC on Polygon -->
                <div class="flex items-center justify-between">
                  <span>USDC(Polygon):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingAvocadoBalance" />
                    <span v-else>{{ usdcOnPolygonOfAvocado }}</span>
                    <span>USDC</span>
                  </span>
                </div>

                <!-- DAI on Arbi -->
                <div class="flex items-center justify-between">
                  <span>DAI(Arbi):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingAvocadoBalance" />
                    <span v-else>{{ daiOnArbiOfAvocado }}</span>
                    <span>DAI</span>
                  </span>
                </div>

                <!-- USDT on Optimism -->
                <div class="flex items-center justify-between">
                  <span>USDT(Optimism):</span>
                  <span class="flex space-x-2 items-center font-bold">
                    <Skeleton v-if="isFetchingAvocadoBalance" />
                    <span v-else>{{ usdtOnOptimismOfAvocado }}</span>
                    <span>UDST</span>
                  </span>
                </div>
              </div>
            </div>
          </div>

          <Button @click="transferUSDC">
            <span class="text-center">
              Transfer USDC on Polygon to Avocado
            </span>
          </Button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ethers } from "ethers";
import usdcABI from "~/src/abi/usdc.json";
import daiABI from "~/src/abi/dai.json";
import usdtABI from "~/src/abi/usdt.json";
import forwarderABI from "~/src/abi/forwarder.json";
import Button from "~/components/Button.vue";
import Skeleton from "~/components/Skeleton.vue";
import { toHex } from "~/utils";

// constrints
const usdcContractAddressPolygon = "0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359";
const daiContractAddressArbi = "0xDA10009cBd5D07dd0CeCc66161FC93D7c9000da1";
const usdtContractAddressOptimism =
  "0x94b008aA00579c1307B0EF2c499aD98a8ce58e58";
const forwarderContractAddress = "0x46978CD477A496028A18c02F07ab7F35EDBa5A54";
const polygonRpcProviderUrl = "https://polygon.llamarpc.com";
const arbiRpcProviderUrl = "https://1rpc.io/arb";
const optimismRpcProviderUrl = "https://1rpc.io/op";
const polygonChainId = 137;

// states
const isConnected = useState<boolean>("connected", () => false);
const isConnecting = useState<boolean>("connecting", () => false);
const isFetchingEOABalance = useState<boolean>("connecting", () => false);
const isFetchingAvocadoBalance = useState<boolean>("connecting", () => false);
const eoaAddress = useState<string>("eoa-address", () => "");
const avocadoAddress = useState<string>("avocado-address", () => "");
const usdcOnPolygonOfEOA = useState<number | null>(
  "usdc-polygon-eoa",
  () => null
);
const daiOnArbiOfEOA = useState<number | null>("dai-arbi-eoa", () => null);
const usdtOnOptimismOfEOA = useState<number | null>(
  "usdt-optimism-eoa",
  () => null
);
const usdcOnPolygonOfAvocado = useState<number | null>(
  "usdc-polygon-avocado",
  () => null
);
const daiOnArbiOfAvocado = useState<number | null>(
  "dai-arbi-avocado",
  () => null
);
const usdtOnOptimismOfAvocado = useState<number | null>(
  "usdt-optimism-avocado",
  () => null
);

/**
 * Check if it has already connected
 */
onMounted(async () => {
  if (window.ethereum) {
    try {
      isConnecting.value = true;

      const accounts: string[] = await window.ethereum.request({
        method: "eth_accounts",
      });

      if (accounts.length) {
        isConnected.value = true;

        eoaAddress.value = accounts[0];
      } else {
        isConnected.value = false;
      }

      isConnecting.value = false;
    } catch (error) {
      isConnected.value = false;
      isConnecting.value = false;
    }
  }
});

/**
 * Check wallet is connected or not
 */
watch(isConnected, async (newValue, oldValue) => {
  if (newValue) {
    fetchEOABalance();

    await createAvocadoWallet(eoaAddress.value);

    if (avocadoAddress.value) {
      fetchAvocadoBalance();
    }
  }
});

/**
 * Connect Metamask
 */
const connectWallet = async () => {
  // connect metamask
  if (!import.meta.env.SSR) {
    if (window.ethereum) {
      try {
        isConnecting.value = true;

        const provider = new ethers.providers.Web3Provider(window.ethereum);

        await provider.send("eth_requestAccounts", []);

        const signer = provider.getSigner();

        const address = await signer.getAddress();

        isConnected.value = true;
        isConnecting.value = false;
        eoaAddress.value = address;
      } catch (error) {
        isConnected.value = false;
        isConnecting.value = false;
        eoaAddress.value = "";
      }
    }
  }
};

/**
 * Transfer all USDC balance
 */
const transferUSDC = async () => {
  await transfer(usdcContractAddressPolygon, usdcABI, "0.5", polygonChainId);

  fetchEOABalance();
  fetchAvocadoBalance();
};

/**
 * Get balance based on criteria
 */
const getBalance = async (
  target: string,
  abi: ethers.ContractInterface,
  rpcProviderUrl: string,
  wallet: string
) => {
  const provider = new ethers.providers.JsonRpcProvider(rpcProviderUrl);

  const contract = new ethers.Contract(target, abi, provider);

  const balance = await contract.balanceOf(wallet);

  const decimals = await contract.decimals();

  return ethers.utils.formatUnits(balance, decimals);
};

/**
 * Fetch EOA balances
 */
const fetchEOABalance = async () => {
  isFetchingEOABalance.value = true;

  const usdcBalanceOnPolygonOfEOA = await getBalance(
    usdcContractAddressPolygon,
    usdcABI,
    polygonRpcProviderUrl,
    eoaAddress.value
  );

  const daiBalanceOnArbiOfEOA = await getBalance(
    daiContractAddressArbi,
    daiABI,
    arbiRpcProviderUrl,
    eoaAddress.value
  );

  const usdtBalanceOnOptimismOfEOA = await getBalance(
    usdtContractAddressOptimism,
    usdtABI,
    optimismRpcProviderUrl,
    eoaAddress.value
  );

  usdcOnPolygonOfEOA.value = parseFloat(usdcBalanceOnPolygonOfEOA);
  daiOnArbiOfEOA.value = parseFloat(daiBalanceOnArbiOfEOA);
  usdtOnOptimismOfEOA.value = parseFloat(usdtBalanceOnOptimismOfEOA);

  isFetchingEOABalance.value = false;
};

/**
 * Fetch Avocado balances
 */
const fetchAvocadoBalance = async () => {
  isFetchingAvocadoBalance.value = true;

  const usdcBalanceOnPolygonOfAvocado = await getBalance(
    usdcContractAddressPolygon,
    usdcABI,
    polygonRpcProviderUrl,
    avocadoAddress.value
  );

  const daiBalanceOnArbiOfAvocado = await getBalance(
    daiContractAddressArbi,
    daiABI,
    arbiRpcProviderUrl,
    avocadoAddress.value
  );

  const usdtBalanceOnOptimismOfAvocado = await getBalance(
    usdtContractAddressOptimism,
    usdtABI,
    optimismRpcProviderUrl,
    avocadoAddress.value
  );

  usdcOnPolygonOfAvocado.value = parseFloat(usdcBalanceOnPolygonOfAvocado);
  daiOnArbiOfAvocado.value = parseFloat(daiBalanceOnArbiOfAvocado);
  usdtOnOptimismOfAvocado.value = parseFloat(usdtBalanceOnOptimismOfAvocado);

  isFetchingAvocadoBalance.value = false;
};

/**
 * Create avocado wallet
 */
const createAvocadoWallet = async (eoaAddress: string) => {
  const provider = new ethers.providers.JsonRpcProvider(polygonRpcProviderUrl);

  const contract = new ethers.Contract(
    forwarderContractAddress,
    forwarderABI,
    provider
  );

  const avocadoWallet = await contract.computeAvocado(eoaAddress, 0);

  avocadoAddress.value = avocadoWallet as string;
};

/**
 * Transfer fund to avocado wallet
 */
const transfer = async (
  target: string,
  abi: ethers.ContractInterface,
  amount: string,
  tChainId: number
) => {
  if (window.ethereum) {
    let provider = new ethers.providers.Web3Provider(window.ethereum);

    const { chainId } = await provider.getNetwork();

    if (tChainId !== chainId) {
      await switchNetwork(tChainId);

      provider = new ethers.providers.Web3Provider(window.ethereum);
    }

    const contract = new ethers.Contract(target, abi, provider.getSigner());

    const amountInDecimal = ethers.utils.parseUnits(amount, 6);

    const tx = await contract.transfer(avocadoAddress.value, amountInDecimal);

    await tx.wait();
  }
};

/**
 * Switch network
 */
const switchNetwork = async (chainId: number) => {
  if (window.ethereum) {
    try {
      const provider = new ethers.providers.Web3Provider(window.ethereum);

      await provider.send("wallet_switchEthereumChain", [
        {
          chainId: toHex(chainId),
        },
      ]);
    } catch (error: any) {}
  }
};
</script>
