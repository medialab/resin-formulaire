<script lang="ts">
  import Autocomplete from "./Autocomplete.svelte";
  import Markdown from "./Markdown.svelte";
  import Photo from "./Photo.svelte";
  import AdditionalSkills from "./AdditionalSkills.svelte";

  export let apiServer: string;
  export let endpoint = "/api/members/";
  export let success = false;
  export let initialData: Record<string, any> = {};
  export let token: string | null = null;
  export let sendLabel: string = "S'inscrire";

  let formState: Record<string, any> = initialData;

  type LanguageChoice = {
    pt2b: string;
    name: string;
  };

  type SkillChoice = {
    id: number;
    field: string;
    skill: string;
    detail: string;
  };

  type FormField = {
    label: string;
    required: boolean;
    help_text: string;
    choices?: { value: any; display_name: string }[];
  };

  const getFormFields = async (): Promise<Map<string, FormField>> =>
    (
      await (
        await fetch(`${apiServer}${endpoint}`, {
          headers: {
            "Content-Type": "application/json",
          },
          method: "OPTIONS",
        })
      ).json()
    ).actions.POST;

  const [getLanguages, getSkills, getFields] = [
    "languages",
    "skills",
    "fields",
  ].map(
    (choiceModel) => async () =>
      await (
        await fetch(`${apiServer}/api/${choiceModel}/`, {
          headers: {
            "Content-Type": "application/json",
          },
          method: "GET",
        })
      ).json(),
  ) as [
    () => Promise<LanguageChoice[]>,
    () => Promise<SkillChoice[]>,
    () => Promise<SkillChoice[]>,
  ];

  const getChoices = async (field: "languages" | "expertise" | "skills") => {
    switch (field) {
      case "languages":
        return (await getLanguages()).map(({ pt2b, name }) => ({
          value: pt2b,
          label: name,
        }));
      case "expertise":
        return (await getFields()).map(({ id, field }) => ({
          value: id,
          label: field,
        }));
      case "skills":
        return (await getSkills()).map(({ id, field, skill, detail }) => ({
          value: id,
          label: `${field} - ${detail ? `${skill} - ${detail}` : skill}`,
        }));
    }
  };

  let errors: Record<string, Array<string>> = {};

  async function submit(event: Event) {
    let isUpdate = !!initialData.id && !!token;
    errors = {};
    const form = event.target as HTMLFormElement;
    const formData = new FormData(form);
    // remove empty fields
    for (const [key, value] of formData.entries()) {
      if (value === "") {
        formData.delete(key);
      }
    }

    const response = await fetch(
      `${apiServer}${endpoint}${isUpdate ? `${initialData.id}/` : ""}`,
      {
        method: isUpdate ? "PATCH" : "POST",
        body: formData,
        headers: {
          ...(token ? { Authorization: `Token ${token}` } : {}),
        },
      },
    );

    if (response.status === 400) {
      errors = await response.json();
    }

    if (response.status === 403) {
      errors[""] = (await response.json()).detail;
    }

    if (response.status === 201 || response.status === 200) {
      success = true;
    }
  }

  function scroll(node: HTMLElement) {
    node.scrollIntoView({ behavior: "smooth", block: "center" });
  }

  function htmlValidate(e: FocusEvent) {
    if (!e.target) return;
    const input = e.target as HTMLInputElement;
    if (!input.checkValidity()) {
      input.reportValidity();
    }
  }
</script>

<form id="subscribe" on:submit|preventDefault={submit}>
  {#await Promise.all([getFormFields()])}
    <p>Loading...</p>
  {:then [formFields]}
    {#each Object.entries(formFields).filter(([id, details]) => !details.read_only && !id.startsWith("delete_")) as [id, details] (id)}
      <div class="form-group">
        <label for={id}
          >{details.label}
          {#if details.required}*{/if}</label
        >
        {#if id === "gender"}
          <select
            {id}
            name={id}
            class="form-control"
            required={details.required}
            bind:value={formState[id]}
          >
            {#each details.choices as gender (gender.value)}
              <option value={gender.value}>{gender.display_name}</option>
            {/each}
          </select>
        {:else if id === "email"}
          <input
            type="email"
            {id}
            name={id}
            class="form-control"
            required={details.required}
            bind:value={formState[id]}
            on:blur={htmlValidate}
          />
        {:else if id === "photo"}
          <Photo
            {id}
            required={details.required}
            initialData={initialData[id]}
          />
        {:else if id === "languages" || id === "expertise" || id === "skills"}
          {#await getChoices(id) then choices}
            <Autocomplete
              {id}
              name={id}
              {choices}
              bind:selection={formState[id]}
            />
          {/await}
        {:else if id === "long_bio" || id === "publications" || id === "training" || id === "comments"}
          <Markdown
            {id}
            required={details.required}
            bind:value={formState[id]}
          />
        {:else if id === "additional_skills"}
          {#await getChoices("skills") then choices}
            <AdditionalSkills
              {id}
              bind:value={formState[id]}
              {choices}
              add={(value) => (formState["skills"] = [...formState[id], value])}
            />
          {/await}
        {:else if details.type === "boolean"}
          <input type="checkbox" {id} name={id} />
        {:else}
          <input
            type="text"
            {id}
            name={id}
            class="form-control"
            required={details.required}
            bind:value={formState[id]}
          />
        {/if}
        {#if details.help_text}
          <div id="{id}Help" class="form-text text-muted">
            <small>{details.help_text}</small>
          </div>
        {/if}
        {#if errors[id]}
          <div class="invalid-feedback">
            {#each errors[id] as error (error)}
              <small style:color="#B04838" use:scroll>{error}</small>
            {/each}
          </div>
        {/if}
      </div>
    {/each}
    {#if errors[""]}
      <p>
        {errors[""]}
      </p>
    {/if}
    <button type="submit" class="btn btn-primary">{sendLabel}</button>
  {/await}
</form>
