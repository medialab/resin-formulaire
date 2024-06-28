<svelte:options
  customElement={{
    tag: "resin-formulaire-edit",
    shadow: "none",
  }}
/>

<script lang="ts">
  import Form from "./components/Form.svelte";

  export let apiServer = "http://localhost:8000";

  // Get uid and token params
  let success = false;
</script>

{#if !success}
  <Form
    {apiServer}
    bind:success
    endpoint="/api/members/new_auth_link"
    sendLabel="Recevoir le lien"
  />
{:else}
  <p>
    Vous avez reçu un email contenant un lien pour modifier vos informations.
  </p>
{/if}
