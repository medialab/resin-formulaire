<script lang="ts">
  export let id: string;
  export let required = false;
  export let initialData: string | undefined;

  let input: HTMLInputElement;
  let previewURI: string | undefined;

  function updatePreview(e: Event) {
    const input = e.target as HTMLInputElement;
    if (input.files && input.files[0]) {
      let reader = new FileReader();
      reader.onload = (ev) => {
        previewURI = ev.target?.result as string;
      };
      reader.readAsDataURL(input.files[0]);
    }
  }
</script>

<input
  type="file"
  accept="image/jpeg"
  {id}
  name={id}
  class="form-control"
  {required}
  on:change={updatePreview}
  bind:this={input}
/>
{#if initialData && !previewURI}
  <img src={initialData} alt="Profil" style="width: 200px; height: auto;" />
  <input type="checkbox" id="delete_{id}" name="delete_{id}" />
  <label for="delete_{id}">Supprimer ma photo</label>
{/if}
{#if previewURI}
  <div
    style="
    display: inline-block;
    height: 200px;
    background-size: cover;
    width: 200px;
    background-position: center;
    background-image: url({previewURI});
  "
  />
  <button
    on:click|capture|stopImmediatePropagation|stopPropagation|preventDefault={() => {
      input.value = "";
      previewURI = undefined;
    }}
  >
    &times;
  </button>
{/if}
