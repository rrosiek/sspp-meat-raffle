<script lang="ts">
  import { dev } from "$app/environment";

  const bgImg = dev ? "texture_bg.jpg" : "/texture_bg.jpg";
</script>

<header style="background-image: url('{bgImg}')">
  <div class="max-w-6xl mx-auto p-4 xl:w-2/3 flex flex-col items-center">
    <picture>
      <source srcset="/sspp_logo.webp 1x" type="image/webp" />
      <source srcset="/sspp_logo.png 1x" type="image/png" />
      <img
        src="/sspp_logo.png"
        alt="Saints Peter and Paul School"
        title="SSPP Logo"
      />
    </picture>
    <div class="font-display font-medium text-center text-4xl text-mr-gold">
      You're invited to a very special
    </div>
    <picture>
      <source srcset="/title_logo.webp 1x" type="image/webp" />
      <source srcset="/title_logo.png 1x" type="image/png" />
      <img
        src="/title_logo.png"
        alt="Welcome to SSPP 2022 Meat Raffle"
        title="SSPP Raffle Header"
      />
    </picture>
  </div>
</header>

<style>
  header {
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
  }
</style>
