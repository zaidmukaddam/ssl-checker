<script lang="ts">
  import { Spinner } from "flowbite-svelte";

  let hostname = "";
  let isLoading = false;
  let error: string | null = null;

  interface ChainItem {
    commonName: string;
    issuer: string;
    location: string;
    organization: string;
    serialNumber: string;
    signatureAlgorithm: string;
    validFrom: string;
    validTo: string;
    ipAddress: string;
  }

  interface SSLInfo extends ChainItem {
    chain: ChainItem[];
  }

  let sslInfo: SSLInfo = {
    chain: [],
    commonName: "",
    issuer: "",
    location: "",
    organization: "",
    serialNumber: "",
    signatureAlgorithm: "",
    validFrom: "",
    validTo: "",
    ipAddress: "",
  };

  async function fetchSSLData() {
    isLoading = true;
    error = null;

    // Reset the sslInfo object when starting a new search
    sslInfo = {
      chain: [],
      commonName: "",
      issuer: "",
      location: "",
      organization: "",
      serialNumber: "",
      signatureAlgorithm: "",
      validFrom: "",
      validTo: "",
      ipAddress: "",
    };

    try {
      const response = await fetch(
        `https://ssl-checker-zaidmukaddam.koyeb.app/ssl-info?hostname=${hostname}`
      );
      const result = await response.json();
      if (response.ok && result) {
        sslInfo = result;
      } else {
        error =
          result?.message ||
          "We couldn't reslove the IP address of your domain or your domain is using a deprecated version of SSL. Please update your DNS and SSL Certificate to the latest version.";
      }
    } catch (err) {
      console.error("Error fetching SSL data:", err);
      error =
        "We couldn't reslove the IP address of your domain or your domain is using a deprecated version of SSL. Please update your DNS and SSL Certificate to the latest version.";
    } finally {
      isLoading = false;
    }
  }

  function handleCheckSSL() {
    if (!hostname.trim()) {
      // Check if hostname is empty
      error = "Please provide a hostname before checking SSL.";
      return; // Exit the function
    }
    hostname = cleanInputURL(hostname);
    fetchSSLData();
  }

  function parseDate(dateString: string): Date {
    const year = dateString.substring(0, 4);
    const month = dateString.substring(4, 6);
    const day = dateString.substring(6, 8);
    const hour = dateString.substring(8, 10);
    const minute = dateString.substring(10, 12);
    const second = dateString.substring(12, 14);

    // JavaScript months are 0-based (0 = January, 11 = December)
    return new Date(
      Date.UTC(
        Number(year),
        Number(month) - 1,
        Number(day),
        Number(hour),
        Number(minute),
        Number(second)
      )
    );
  }

  function calculateDaysToExpiry(validToDate: string): number {
    const currentDate = new Date();
    const expiryDate = parseDate(validToDate);

    const differenceInTime = expiryDate.getTime() - currentDate.getTime();
    return Math.ceil(differenceInTime / (1000 * 3600 * 24));
  }

  function formatCertDate(dateString: string) {
    const year = dateString.substring(0, 4);
    const month = dateString.substring(4, 6);
    const day = dateString.substring(6, 8);
    const hour = dateString.substring(8, 10);
    const minute = dateString.substring(10, 12);
    const second = dateString.substring(12, 14);

    const date = new Date(
      `${year}-${month}-${day}T${hour}:${minute}:${second}Z`
    );

    // Now you can format the date any way you'd like. Here's one way:
    return date.toLocaleDateString("en-US", {
      year: "numeric",
      month: "long",
      day: "numeric",
    });
  }

  function cleanInputURL(url: string) {
    // Remove http:// or https://
    let cleaned = url.replace(/^https?:\/\//, "");

    // Remove any routes or extra slashes
    cleaned = cleaned.split("/")[0];

    // Remove subdomains, keeping only the domain and TLD
    const parts = cleaned.split(".");
    if (parts.length > 2) {
      cleaned = parts.slice(-2).join(".");
    }

    return cleaned;
  }

  function isHostnameListed(): boolean {
    return sslInfo.commonName === hostname;
  }
</script>

<svelte:head>
  <title>Home</title>
  <meta name="description" content="SSL Checker" />
</svelte:head>

<section class="flex flex-col justify-center items-center flex-grow-6">
	<img src="/ssl-checker-logo.png" alt="SSL Checker" class="h-24 w-24 mb-4" />
	<h1 class="w-full text-4xl font-bold mb-8 mt-4 text-[#ff3e00]">SSL Checker</h1>
	<div class="bg-white shadow-xl p-6 rounded-lg w-full relative welcome">
	  <input
		bind:value={hostname}
		placeholder="Enter Server Hostname..."
		class="p-3 w-full rounded-lg border dark:border-gray-700 dark:bg-gray-900 dark:text-gray-300 mb-4 transition-all duration-200 hover:border-[#ff3e00] focus:border-[#ff3e00] focus:outline-none"
	  />
	  <button
		on:click={handleCheckSSL}
		class="w-full bg-[#ff3e00] hover:bg-[#e83500] text-white p-3 rounded-lg mb-4 transition-all duration-200 transform hover:scale-[1]"
	  >
		Check SSL
	  </button>
	  {#if error}
		<div class="text-red-500 mb-4 border-l-4 border-red-500 pl-3">{error}</div>
	  {/if}
	  {#if isLoading}
		<div class="flex justify-center items-center h-40">
		  <Spinner color="blue" />
		</div>
	  {:else if sslInfo.commonName}
		<div class="bg-[#4075a6] p-6 rounded-lg text-white shadow-lg w-full">
        <div class="flex items-center mb-4">
          <img src="/server-ok.png" alt="Server OK" class="h-12 w-12 mr-4" />
          <div>
            <h2 class="text-2xl font-bold mb-2">{sslInfo.commonName}</h2>
            <p class="mb-1">
              {sslInfo.commonName} resolves to {sslInfo.ipAddress}
            </p>
            <p class="mb-1">
              Certificate expires in {calculateDaysToExpiry(sslInfo.validTo)} days
            </p>
            <p>
              Hostname is correctly listed in the certificate: {isHostnameListed()
                ? "Yes"
                : "No"}
            </p>
            <p>
              <span class="font-semibold">Valid From:</span>
              {formatCertDate(sslInfo.validFrom)} to {formatCertDate(
                sslInfo.validTo
              )}
            </p>
            <p>
              <span class="font-semibold">Serial Number:</span>
              {sslInfo.serialNumber}
            </p>
            <p>
              <span class="font-semibold">Signature Algorithm:</span>
              {sslInfo.signatureAlgorithm}
            </p>
          </div>
        </div>
        {#each sslInfo.chain as chainItem, index}
          <div class="p-4 border-t border-gray-600">
            <img src="/chain-ok.png" alt="Server OK" class="h-12 w-12 mr-4" />
            <h3 class="text-blue-400 mb-2">Chain {index + 1}</h3>
            <p class="mb-1">
              <span class="font-semibold">Common Name:</span>
              {chainItem.commonName}
            </p>
            <p class="mb-1">
              <span class="font-semibold">Organization:</span>
              {chainItem.organization}
            </p>
            <p class="mb-1">
              <span class="font-semibold">Location:</span>
              {chainItem.location}
            </p>
            <p class="mb-1">
              <span class="font-semibold">Valid From:</span>
              {formatCertDate(chainItem.validFrom)} to {formatCertDate(
                chainItem.validTo
              )}
            </p>
            <p class="mb-1">
              <span class="font-semibold">Serial Number:</span>
              {chainItem.serialNumber}
            </p>
            <p class="mb-1">
              <span class="font-semibold">Signature Algorithm:</span>
              {chainItem.signatureAlgorithm}
            </p>
            <p><span class="font-semibold">Issuer:</span> {chainItem.issuer}</p>
          </div>
        {/each}
      </div>
    {/if}
  </div>
</section>
