<svelte:options
  customElement={{
    tag: "resin-formulaire",
    shadow: "none",
  }}
/>

<script lang="ts">
  import Form from "./components/Form.svelte";

  export let apiServer = window.location.origin;

  // Get uid and token params
  const urlParams = new URLSearchParams(window.location.search);
  const uid = urlParams.get("uid");
  const token = urlParams.get("token");

  let success = false;

  const getData = async () => {
    if (!uid) return {};

    let res;
    try {
      res = await fetch(`${apiServer}/api/members/${uid}/`, {
        headers: {
          "Content-Type": "application/json",
          ...(token ? { Authorization: `Token ${token}` } : {}),
        },
        method: "GET",
      });
    } catch (error) {
      throw new Error("Impossible de récupérer les données.");
    }

    if (res.status === 404) {
      throw new Error("Cet identifiant de membre n'existe pas.");
    }

    if (res.status === 403) {
      throw new Error("Ce token est invalide ou a expiré.");
    }

    if (res.status === 200) {
      return await res.json();
    }
  };
</script>

{#if !success}
  {#await getData()}
    <p>Chargement...</p>
  {:then data}
    <Form
      bind:success
      initialData={data}
      {token}
      {apiServer}
      sendLabel={uid ? "Mettre à jour" : "S'inscrire"}
    />
  {:catch error}
    <p>Une erreur est survenue. {error.message}</p>
  {/await}
{:else}
  <p>
    Vos informations ont bien été transmises.
    {#if token}
      Votre profil doit encore être validé par l'administratrice avant d'être
      publié.
    {/if}
  </p>
{/if}
