<script lang="ts">
  import { onMount } from "svelte";
  import { dev } from "$app/environment";
  import { initFirebase } from "$lib/firebase";
  import "../app.css";

  onMount(async () => {
    const src = dev
      ? "https://sandbox.web.squarecdn.com/v1/square.js"
      : "https://web.squarecdn.com/v1/square.js";
    const scriptEl = document.createElement("script");

    scriptEl.setAttribute("type", "text/javascript");
    scriptEl.setAttribute("src", src);
    document.body.appendChild(scriptEl);
  });

  initFirebase({
    apiKey: import.meta.env.VITE_FIREBASE_API_KEY as string,
    authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN as string,
    projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID as string,
    storageBucket: import.meta.env.VITE_FIREBASE_STORAGE_BUCKET as string,
    messagingSenderId: import.meta.env
      .VITE_FIREBASE_MESSAGING_SENDING_ID as string,
    appId: import.meta.env.VITE_FIREBASE_APP_ID as string,
    measurementId: import.meta.env.VITE_FIREBASE_MEASUREMENT_ID as string,
  });
</script>

<slot />
