<script lang="ts">
  import { marked } from "marked";
  import DOMPurify from "dompurify";

  export let id: string;
  export let value = "";
  export let required = false;

  let preview = "";
  let showPreview = false;

  $: (async () => {
    preview = DOMPurify.sanitize(await marked.parse(value));
  })();
</script>

<textarea {id} name={id} class="form-control" {required} bind:value></textarea>
<div class="form-text text-muted">
  <small>
    Ce champ permet le formatage <a
      href="https://fr.wikipedia.org/wiki/Markdown#Exemples_de_la_syntaxe"
      >Markdown</a
    >.
    {#if value}
      <button
        title="Prévisualiser"
        on:click|preventDefault|stopPropagation={() =>
          (showPreview = !showPreview)}
        >{#if !showPreview}Prévisualiser{:else}Fermer{/if}</button
      >
    {/if}
  </small>
</div>
{#if showPreview}
  <div>{@html preview}</div>
{/if}
