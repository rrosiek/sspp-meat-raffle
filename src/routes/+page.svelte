<script lang="ts">
  import { getFirebaseApp } from "$lib/firebase";
  import EventDetail from "$lib/components/EventDetail.svelte";
  import Header from "$lib/components/Header.svelte";
  import Payment from "$lib/components/Payment.svelte";
  import ThankYou from "$lib/components/ThankYou.svelte";

  const app = getFirebaseApp();

  let paymentEmail = null;

  const handlePaymentComplete = (event) => (paymentEmail = event.detail.email);
</script>

<Header />

<main>
  <EventDetail />

  {#if app && !paymentEmail}
    <Payment on:paymentComplete={handlePaymentComplete} />
  {:else if paymentEmail}
    <ThankYou email={paymentEmail} />
  {/if}
</main>
