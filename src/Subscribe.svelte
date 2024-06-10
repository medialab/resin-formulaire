<svelte:options
  customElement={{
    tag: "resin-subscribe",
    shadow: "none",
    reflect: true,
  }}
/>

<script lang="ts">
  import SubscribeForm from "./components/SubscribeForm.svelte";

  let success = false;

  console.log("Subscribe");
</script>

{#if !success}
  <SubscribeForm bind:success />
{:else}
  <p>
    Vos informations ont bien été transmises. Votre profil doit encore être
    validé par l'administratrice avant d'être publié.
  </p>
{/if}
