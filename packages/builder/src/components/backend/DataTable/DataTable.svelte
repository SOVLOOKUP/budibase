<script>
  import { tables, views } from "stores/backend"

  import CreateRowButton from "./buttons/CreateRowButton.svelte"
  import CreateColumnButton from "./buttons/CreateColumnButton.svelte"
  import CreateViewButton from "./buttons/CreateViewButton.svelte"
  import ExportButton from "./buttons/ExportButton.svelte"
  import EditRolesButton from "./buttons/EditRolesButton.svelte"
  import ManageAccessButton from "./buttons/ManageAccessButton.svelte"
  import HideAutocolumnButton from "./buttons/HideAutocolumnButton.svelte"
  import * as api from "./api"
  import Table from "./Table.svelte"
  import { TableNames } from "constants"
  import CreateEditUser from "./modals/CreateEditUser.svelte"
  import CreateEditRow from "./modals/CreateEditRow.svelte"

  let hideAutocolumns = true
  let data = []
  let loading = false
  $: isUsersTable = $tables.selected?._id === TableNames.USERS
  $: title = $tables.selected.name
  $: schema = $tables.selected.schema
  $: tableView = {
    schema,
    name: $views.selected?.name,
  }

  // Fetch rows for specified table
  $: {
    if ($views.selected?.name?.startsWith("all_")) {
      loading = true
      const loadingTableId = $tables.selected?._id
      api.fetchDataForView($views.selected).then(rows => {
        loading = false

        // If we started a slow request then quickly change table, sometimes
        // the old data overwrites the new data.
        // This check ensures that we don't do that.
        if (loadingTableId !== $tables.selected?._id) {
          return
        }

        data = rows || []
      })
    }
  }
</script>

<Table
  {title}
  {schema}
  tableId={$tables.selected?._id}
  {data}
  allowEditing={true}
  bind:hideAutocolumns
  {loading}
>
  <CreateColumnButton />
  {#if schema && Object.keys(schema).length > 0}
    <CreateRowButton
      title={isUsersTable ? "Create user" : "Create row"}
      modalContentComponent={isUsersTable ? CreateEditUser : CreateEditRow}
    />
    <CreateViewButton />
    <ManageAccessButton resourceId={$tables.selected?._id} />
    {#if isUsersTable}
      <EditRolesButton />
    {/if}
    <HideAutocolumnButton bind:hideAutocolumns />
    <!-- always have the export last -->
    <ExportButton view={tableView} />
  {/if}
</Table>
