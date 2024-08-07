<script lang="ts">
  type Choice = {
    value: any;
    label: string;
  };

  export let choices: Choice[] = [];
  export let name: string;
  export let id: string;
  export let selection: Array<any> = [];

  let componentElement: HTMLDivElement;
  let focus = false;
  let search = "";
  let searchResult: Choice[] = [];

  $: searchResult = choices.filter(
    (choice) =>
      choice.label.toLowerCase().includes(search.toLowerCase()) &&
      !selection.includes(choice.value),
  );

  function focusOut(event: FocusEvent | MouseEvent) {
    if (
      !componentElement.contains((event.target || event.relatedTarget) as Node)
    ) {
      focus = false;
    }
  }

  function inputKeydown(event: KeyboardEvent) {
    if (event.key === "ArrowDown") {
      event.preventDefault();
      const target = event.target as HTMLInputElement;
      const button = target.nextElementSibling?.querySelector("button");
      if (button) {
        button.focus();
      }
    }
  }

  function listKeydown(event: KeyboardEvent) {
    if (event.key === "ArrowDown") {
      event.preventDefault();
      const target = event.target as HTMLButtonElement;
      const next =
        target.parentElement?.nextElementSibling?.querySelector("button");
      next?.focus();
    }

    if (event.key === "ArrowUp") {
      event.preventDefault();
      const target = event.target as HTMLButtonElement;
      const prev =
        target.parentElement?.previousElementSibling?.querySelector("button");
      prev?.focus();
    }

    // Echap
    if (event.key === "Escape") {
      event.preventDefault();
      focus = false;
    }
  }

  function addSelection(choice: Choice) {
    if (selection.length >= 6) {
      return;
    }
    selection = [...selection, choice.value];
    search = "";
    focus = true;
  }

  window.addEventListener("click", focusOut);
</script>

<div bind:this={componentElement}>
  {#each selection as value (value)}
    <input type="hidden" {name} {value} />
  {/each}
  <input
    type="text"
    {id}
    placeholder="Tapez pour chercher..."
    bind:value={search}
    on:focus={() => (focus = true)}
    on:blur={focusOut}
    on:focusout={focusOut}
    on:keydown={inputKeydown}
  />
  {#if focus}
    <ul class="choices">
      {#each searchResult as choice (choice.value)}
        <li>
          <button
            on:click|stopPropagation|preventDefault={() => addSelection(choice)}
            on:focusout={focusOut}
            on:keydown={listKeydown}
          >
            {choice.label}
          </button>
        </li>
      {/each}
    </ul>
  {/if}
  {#if selection.length > 0}
    <ul class="selection">
      {#each selection as value (value)}
        <li>
          <button
            on:click|stopPropagation|preventDefault={() =>
              (selection = selection.filter((v) => v !== value))}
          >
            &times;
          </button>
          {choices.find((choice) => choice.value === value)?.label}
        </li>
      {/each}
    </ul>
  {/if}
  {#if id === "skills"}
    <div id="{id}Help" class="form-text text-muted">
      <small
        >Vous pouvez ajouter encore {6 - selection.length} compétences.
      </small>
    </div>
  {/if}
</div>

<style>
  /* Idéalement tout le css devrait être dans le projet affichant le formulaire */
  ul.selection {
    list-style-type: none;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
  }

  ul.selection li {
    margin-right: 1rem;
  }

  ul.selection button {
    border: 0;
    background-color: transparent;
    cursor: pointer;
    height: 1.5rem;
    width: 1.5rem;
    color: #666;

    &:hover {
      background-color: #f9f9f9;
    }
  }

  ul.choices {
    list-style-type: none;
    padding: 0;
    margin: 0;
    border: 1px solid #ccc;
    max-height: 200px;
    overflow-y: auto;
  }

  ul.choices li {
    padding: 0;
    cursor: pointer;
  }

  ul.choices li:hover {
    background-color: #f9f9f9;
  }

  ul.choices button {
    text-align: left;
    border: 0;
    padding: 0.5rem;
    width: 100%;
  }
</style>
