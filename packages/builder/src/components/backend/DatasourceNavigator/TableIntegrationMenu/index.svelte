<script>
  import { Body } from "@budibase/bbui"
  import { onMount } from "svelte"
  import api from "builderStore/api"
  import ICONS from "../icons"

  export let integration = {}

  let schema
  let integrations = []

  async function fetchIntegrations() {
    const response = await api.get("/api/integrations")
    const json = await response.json()

    integrations = json
    return json
  }

  function selectIntegration(integrationType) {
    schema = integrations[integrationType].datasource
    integration = {
      type: integrationType,
      ...Object.keys(schema).reduce(
        (acc, next) => ({ ...acc, [next]: schema[next].default }),
        {}
      ),
    }
  }

  onMount(() => {
    fetchIntegrations()
  })
</script>

<section>
  <div class="integration-list">
    {#each Object.keys(integrations) as integrationType}
      <div
        class="integration hoverable"
        class:selected={integration.type === integrationType}
        on:click={() => selectIntegration(integrationType)}
      >
        <svelte:component
          this={ICONS[integrationType]}
          height="50"
          width="50"
        />
        <Body size="XS">{integrationType}</Body>
      </div>
    {/each}
  </div>
</section>

<style>
  .integration-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    grid-gap: var(--spectrum-alias-grid-baseline);
  }

  .integration {
    display: grid;
    background: var(--background-alt);
    place-items: center;
    grid-gap: var(--spectrum-alias-grid-margin-xsmall);
    padding: var(--spectrum-alias-item-padding-s);
    transition: 0.3s all;
    border-radius: var(--spectrum-alias-item-rounded-border-radius-s);
  }

  .integration:hover,
  .selected {
    background: var(--spectrum-alias-background-color-tertiary);
  }
</style>
