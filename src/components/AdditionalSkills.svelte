<script lang="ts">
  import Fuse, { type FuseResult } from "fuse.js";

  export let id: string;
  export let value: string = "";
  export let choices: { value: number; label: string }[];
  export let add: (value: number) => void;

  let match: FuseResult<{ value: number; label: string }>[];

  const fuse = new Fuse(choices, {
    keys: ["label"],
    minMatchCharLength: 3,
    threshold: 0.3,
  });

  $: match = fuse.search(value, {
    limit: 4,
  });
</script>

<input type="text" {id} name={id} class="form-control" bind:value />
{#if match?.length}
  <div class="form-text text-danger">
    Les compétences existantes suivantes existent déjà et peuvent être ajoutées
    directement : {#each match as { item }, i (item.value)}
      <a
        href="#"
        on:click|preventDefault|stopPropagation={() => {
          add(item.value);
          value = "";
        }}>{item.label}</a
      >
      {#if i < match.length - 2},
      {:else if i < match.length - 1}
        et
      {:else}.{/if}
    {/each}
  </div>
{/if}
