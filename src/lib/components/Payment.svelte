<script lang="ts">
  import { onMount, createEventDispatcher } from "svelte";
  import * as yup from "yup";
  import { saveRafflePurchase } from "$lib/firebase";
  import Form from "$lib/components/Form.svelte";
  import states from "$lib/states";

  const squareAppId = import.meta.env.VITE_SQUARE_APP_ID;
  const squareLocationId = import.meta.env.VITE_SQUARE_LOCATION_ID;
  const ticketCost = 1000;
  let card = null;
  let cardError = false;
  let ccDonate = false;
  let loading = false;
  let ticketCount = 1;

  $: ticketCostTotal =
    ticketCount * ticketCost +
    (ccDonate ? 200 : 0) -
    (ticketCount === 8 ? 500 : 0);

  const dispatch = createEventDispatcher();

  const moneyFormatter = new Intl.NumberFormat("en-US", {
    style: "currency",
    currency: "USD",
    minimumFractionDigits: 2,
  });

  const schema = yup.object().shape({
    ticketsPurchased: yup.number().integer().required().positive().lessThan(9),
    name: yup.string().required().min(3).max(250).trim(),
    address: yup.string().required().min(3).max(250).trim(),
    city: yup.string().required().min(3).max(250).trim(),
    state: yup.mixed().oneOf(states.map((s) => s.value)),
    zipCode: yup.string().matches(/[0-9]{5}/),
    email: yup.string().required().email(),
    phone: yup.string().matches(/^\(?[0-9]{3}\)?-?\s?[0-9]{3}-?[0-9]{4}$/),
  });

  const cardStyle = {
    ".input-container": {
      borderColor: "#2D2D2D",
      borderRadius: "2px",
    },
    ".input-container.is-focus": {
      borderColor: "#c39b2d",
    },
    ".input-container.is-error": {
      borderColor: "#ff1600",
    },
    ".message-text": {
      color: "#dadada",
    },
    ".message-icon": {
      color: "#dadada",
    },
    ".message-text.is-error": {
      color: "#ff1600",
    },
    ".message-icon.is-error": {
      color: "#ff1600",
    },
    input: {
      fontFamily: "futura, helvetica neue, sans-serif",
    },
    "input::placeholder": {
      color: "#999999",
    },
    "input.is-error": {
      color: "#ff1600",
    },
  };

  const initPayment = async () => {
    const payments = (window as any).Square.payments(
      squareAppId,
      squareLocationId
    );

    card = await payments.card({ style: cardStyle });
    await card.attach("#sq-card");
  };

  const tokenize = async (paymentMethod) => {
    const tokenResult = await paymentMethod.tokenize();

    if (tokenResult.status === "OK") {
      return tokenResult.token;
    } else {
      let errorMessage = `Tokenization failed with status: ${tokenResult.status}`;

      if (tokenResult.errors) {
        errorMessage += ` and errors: ${JSON.stringify(tokenResult.errors)}`;
      }

      throw new Error(errorMessage);
    }
  };

  const handleSubmit = async (event) => {
    loading = true;

    const token = await tokenize(card);

    await saveRafflePurchase({
      ...event.detail,
      ccDonate,
      squarePurchase: { sourceId: token, locationId: squareLocationId },
    });

    dispatch("paymentComplete", event.detail);
    loading = false;
  };

  const handleTicketIncrease = () => {
    if (ticketCount < 8) ticketCount = ticketCount + 1;
  };

  const handleTicketDecrease = () => {
    if (ticketCount > 1) ticketCount = ticketCount - 1;
  };

  onMount(async () => {
    if (!(window as any).Square) {
      throw new Error("Square.js failed to load properly");
    }

    await initPayment();
  });
</script>

<section class="bg-mr-green text-gray-100">
  <div class="max-w-6xl mx-auto py-6 px-4 lg:py-8 xl:w-2/3">
    <div
      class="font-display font-medium tracking-wide text-4xl text-center text-mr-gold uppercase mb-3"
    >
      Purchase Tickets
    </div>
  </div>

  <!-- <div class="text-center text-xl">Online sales available soon!</div> -->
  <Form on:validated={handleSubmit} {schema} let:hasError>
    <div class="max-w-3xl mx-auto p-3">
      <div class="max-w-xl mx-auto">
        <div class="flex items-start">
          <button
            on:click={handleTicketDecrease}
            class="inline-flex justify-center items-center py-3 px-8 bg-mr-gold hover:bg-yellow-500 focus:outline-none focus:border-yellow-700 focus:shadow-outline-indigo active:bg-yellow-700 transition ease-in-out duration-150 rounded-md shadow-md"
            type="button"
            ><svg
              class="w-8 h-8 text-gray-100"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
              ><path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M20 12H4"
              /></svg
            ></button
          >
          <div class="flex-1 mx-4 sm:mx-6">
            <div class="mb-2 rounded-sm shadow-sm">
              <input
                bind:value={ticketCount}
                id="ticketsPurchased"
                name="ticketsPurchased"
                class="text-center text-3xl text-gray-800 form-input px-2 py-3 block w-full border rounded-sm focus:border-mr-gold transition duration-150 ease-in-out sm:leading-5"
                type="number"
                min="1"
                max="8"
              />
            </div>
            <label
              for="ticketsPurchased"
              class="block ml-2 text-lg text-center opacity-75 leading-5 uppercase"
            >
              Number of Tickets
            </label>
            <div
              class="ml-2 text-red-500 text-sm {hasError('ticketsPurchased')
                ? 'block'
                : 'hidden'}"
            >
              Please enter a valid ticket number.
            </div>
          </div>
          <button
            on:click={handleTicketIncrease}
            class="flex-shrink inline-flex justify-center items-center py-3 px-8 bg-mr-gold hover:bg-yellow-500 focus:outline-none focus:border-yellow-700 focus:shadow-outline-green active:bg-yellow-700 transition ease-in-out duration-150 rounded-md shadow-md"
            type="button"
            ><svg
              class="w-8 h-8 text-gray-100"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
              xmlns="http://www.w3.org/2000/svg"
              ><path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M12 6v6m0 0v6m0-6h6m-6 0H6"
              /></svg
            ></button
          >
        </div>
      </div>
      <div class="my-4">
        <div class="mb-2 rounded-sm shadow-sm">
          <input
            id="name"
            name="name"
            class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
          />
        </div>
        <label
          for="name"
          class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
        >
          Name
        </label>
        <div
          class="ml-2 text-red-500 text-sm {hasError('name')
            ? 'block'
            : 'hidden'}"
        >
          Your name is required.
        </div>
      </div>
      <div class="my-4">
        <div class="mb-2 rounded-sm shadow-sm">
          <input
            id="address"
            name="address"
            class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
          />
        </div>
        <label
          for="address"
          class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
        >
          Mailing Street Address
        </label>
        <div
          class="ml-2 text-red-500 text-sm {hasError('address')
            ? 'block'
            : 'hidden'}"
        >
          Your mailing address is required.
        </div>
      </div>
      <div class="my-4 grid grid-cols-6 gap-4">
        <div class="col-span-6 sm:col-span-3">
          <div class="mb-2 rounded-sm shadow-sm">
            <input
              id="city"
              name="city"
              class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
            />
          </div>
          <label
            for="city"
            class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
          >
            City
          </label>
          <div
            class="ml-2 text-red-500 text-sm {hasError('city')
              ? 'block'
              : 'hidden'}"
          >
            Your city is required.
          </div>
        </div>
        <div class="col-span-3 sm:col-span-2">
          <div class="mb-2 rounded-sm shadow-sm">
            <select
              id="state"
              name="state"
              class="text-gray-800 form-select px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
            >
              {#each states as { label, value }}
                <option {value} selected={value === "NY"}>
                  {label.toUpperCase()}
                </option>
              {/each}
            </select>
          </div>
          <label
            for="state"
            class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
          >
            State
          </label>
          <div
            class="ml-2 text-red-500 text-sm {hasError('state')
              ? 'block'
              : 'hidden'}"
          >
            State is required.
          </div>
        </div>
        <div class="col-span-3 sm:col-span-1">
          <div class="mb-2 rounded-sm shadow-sm">
            <input
              id="zipCode"
              name="zipCode"
              class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
              type="number"
              maxlength="5"
            />
          </div>
          <label
            for="state"
            class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
          >
            Zip Code
          </label>
          <div
            class="ml-2 text-red-500 text-sm {hasError('zipCode')
              ? 'block'
              : 'hidden'}"
          >
            Zip code is required.
          </div>
        </div>
      </div>
      <div class="my-4 grid grid-cols-2 gap-4">
        <div class="col-span-2 sm:col-span-1">
          <div class="mb-2 rounded-sm shadow-sm">
            <input
              id="email"
              name="email"
              class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
              type="email"
            />
          </div>
          <label
            for="email"
            class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
          >
            E-Mail Address
          </label>
          <div
            class="mb-2 text-red-500 text-sm {hasError('email')
              ? 'block'
              : 'hidden'}"
          >
            Valid e-mail address is required.
          </div>
        </div>
        <div class="col-span-2 sm:col-span-1">
          <div class="mb-2 rounded-sm shadow-sm">
            <input
              id="phone"
              name="phone"
              placeholder="716-555-1234"
              class="text-gray-800 form-input px-2 py-3 block w-full border rounded-sm text-lg focus:border-mr-gold transition duration-150 ease-in-out"
              type="tel"
            />
          </div>
          <label
            for="phone"
            class="block ml-2 text-white text-lg opacity-75 leading-5 uppercase"
          >
            Phone Number
          </label>
          <div
            class="ml-2 text-red-500 text-sm {hasError('phone')
              ? 'block'
              : 'hidden'}"
          >
            Valid phone number address is required.
          </div>
        </div>
      </div>
      <div class="flex justify-between items-center mt-8 mb-2 sm:mb-4">
        <h3 class="text-white text-xl opacity-75 leading-5 uppercase">
          Billing
        </h3>
        <div class="text-sm text-white opacity-75">
          Processing by
          <a
            href="https://squareup.com"
            target="_blank"
            class="text-mr-gold hover:text-yellow-500">Square</a
          >
        </div>
      </div>
      <div id="sq-card" />
      <div class="text-red-500 text-sm {cardError ? 'block' : 'hidden'}">
        There was a problem processing your card, please try again.
      </div>
      <div class="flex items-center my-6">
        <input
          bind:checked={ccDonate}
          id="ccDonate"
          name="ccDonate"
          type="checkbox"
          class="form-checkbox h-6 w-6 text-mr-gold transition duration-150 ease-in-out"
        />
        <label
          for="donate"
          class="ml-2 block text-sm sm:text-lg text-white opacity-75"
        >
          Donate
          <span class="font-bold">$2.00</span>
          to help cover credit card processing charges.
        </label>
      </div>
    </div>

    <div class="max-w-lg mx-auto px-2">
      {#if !loading}
        <button
          type="submit"
          class="inline-flex items-center justify-center w-full my-6 px-4 py-3 shadow font-display text-2xl font-semibold uppercase tracking-wider bg-mr-gold hover:bg-yellow-500 focus:outline-none focus:border-yellow-600 focus:shadow-outline-red active:bg-yellow-700 transition ease-in-out duration-150"
        >
          <div>Pay {moneyFormatter.format(ticketCostTotal / 100)}</div>
        </button>
      {:else}
        <div
          class="inline-flex items-center justify-center w-full my-6 px-4 py-3.5 shadow font-display text-2xl font-semibold bg-mr-gold hover:bg-yellow-600 uppercase tracking-wider focus:outline-none focus:border-yellow-700 focus:shadow-outline-red active:bg-red-700 transition ease-in-out duration-150"
        >
          <svg class="animate-spin h-8 w-8" viewBox="0 0 24 24">
            <circle
              class="opacity-25"
              cx="12"
              cy="12"
              r="10"
              fill="none"
              stroke="currentColor"
              stroke-width="4"
            />
            <path
              class="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
          </svg>
        </div>
      {/if}
    </div>
  </Form>
</section>
