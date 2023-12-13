<template>
  <h1>Index page</h1>
</template>

<script setup lang="ts">
import { ethers } from "ethers";
import usdcAbi from "~/src/abi/usdc.json";

// const usdcContractAddressPolygon = "0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359";

// const provider = new ethers.JsonRpcProvider("https://polygon.llamarpc.com");

// const usdcContract = new ethers.Contract(
//   usdcContractAddressPolygon,
//   usdcAbi,
//   provider
// );

// connect metamask
if (!import.meta.env.SSR) {
  if (window.ethereum === null) {
    console.log("MetaMask not installed; using read-only defaults");
  } else {
    const ethProvider = new ethers.BrowserProvider(window.ethereum);

    const signer = await ethProvider.getSigner();

    const eoaAddress = await signer.getAddress();

    console.log(eoaAddress);
  }
}
</script>
