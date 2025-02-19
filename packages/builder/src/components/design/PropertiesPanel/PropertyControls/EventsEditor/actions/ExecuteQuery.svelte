<script>
  import { Select, Label, Layout } from "@budibase/bbui"
  import { store, currentAsset } from "builderStore"
  import { datasources, integrations, queries } from "stores/backend"
  import { getBindableProperties } from "builderStore/dataBinding"
  import ParameterBuilder from "components/integration/QueryParameterBuilder.svelte"
  import IntegrationQueryEditor from "components/integration/index.svelte"

  export let parameters

  $: query = $queries.list.find(q => q._id === parameters.queryId)
  $: datasource = $datasources.list.find(
    ds => ds._id === parameters.datasourceId
  )
  $: bindableProperties = getBindableProperties(
    $currentAsset,
    $store.selectedComponentId
  )

  function fetchQueryDefinition(query) {
    const source = $datasources.list.find(ds => ds._id === query.datasourceId)
      .source
    return $integrations[source].query[query.queryVerb]
  }
</script>

<Layout noGap noPadding>
  <Label>Datasource</Label>
  <Select
    bind:value={parameters.datasourceId}
    option={$datasources.list}
    getOptionLabel={source => source.name}
    getOptionValue={source => source._id}
  />

  {#if parameters.datasourceId}
    <Label>Query</Label>
    <Select
      bind:value={parameters.queryId}
      options={$queries.list.filter(
        query => query.datasourceId === datasource._id
      )}
      getOptionLabel={query => query.name}
      getOptionValue={query => query._id}
    />
  {/if}

  {#if query?.parameters?.length > 0}
    <ParameterBuilder
      bind:customParams={parameters.queryParams}
      parameters={query.parameters}
      bindings={bindableProperties}
    />
    <IntegrationQueryEditor
      height={200}
      {query}
      schema={fetchQueryDefinition(query)}
      editable={false}
    />
  {/if}
</Layout>
