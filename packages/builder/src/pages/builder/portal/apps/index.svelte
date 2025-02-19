<script>
  import {
    Heading,
    Layout,
    Button,
    ActionButton,
    ActionGroup,
    ButtonGroup,
    Select,
    Modal,
    ModalContent,
    Page,
    notifications,
    Body,
  } from "@budibase/bbui"
  import CreateAppModal from "components/start/CreateAppModal.svelte"
  import api, { del } from "builderStore/api"
  import analytics from "analytics"
  import { onMount } from "svelte"
  import { apps } from "stores/portal"
  import download from "downloadjs"
  import { goto } from "@roxi/routify"
  import ConfirmDialog from "components/common/ConfirmDialog.svelte"
  import AppCard from "components/start/AppCard.svelte"
  import AppRow from "components/start/AppRow.svelte"

  let layout = "grid"
  let template
  let appToDelete
  let creationModal
  let deletionModal
  let creatingApp = false
  let loaded = false

  const checkKeys = async () => {
    const response = await api.get(`/api/keys/`)
    const keys = await response.json()
    if (keys.userId) {
      analytics.identify(keys.userId)
    }
  }

  const initiateAppCreation = () => {
    creationModal.show()
    creatingApp = true
  }

  const initiateAppImport = () => {
    template = { fromFile: true }
    creationModal.show()
    creatingApp = true
  }

  const stopAppCreation = () => {
    template = null
    creatingApp = false
  }

  const openApp = app => {
    $goto(`../../app/${app._id}`)
  }

  const exportApp = app => {
    try {
      download(
        `/api/backups/export?appId=${app._id}&appname=${encodeURIComponent(
          app.name
        )}`
      )
      notifications.success("App export complete")
    } catch (err) {
      console.error(err)
      notifications.error("App export failed")
    }
  }

  const deleteApp = app => {
    appToDelete = app
    deletionModal.show()
  }

  const confirmDeleteApp = async () => {
    if (!appToDelete) {
      return
    }
    await del(`/api/applications/${appToDelete?._id}`)
    await apps.load()
    appToDelete = null
  }

  onMount(async () => {
    checkKeys()
    await apps.load()
    loaded = true
  })
</script>

<Page wide>
  {#if $apps.length}
    <Layout noPadding>
      <div class="title">
        <Heading>Apps</Heading>
        <ButtonGroup>
          <Button secondary on:click={initiateAppImport}>Import app</Button>
          <Button cta on:click={initiateAppCreation}>Create new app</Button>
        </ButtonGroup>
      </div>
      <div class="filter">
        <div class="select">
          <Select quiet placeholder="Filter by groups" />
        </div>
        <ActionGroup>
          <ActionButton
            on:click={() => (layout = "grid")}
            selected={layout === "grid"}
            quiet
            icon="ClassicGridView"
          />
          <ActionButton
            on:click={() => (layout = "table")}
            selected={layout === "table"}
            quiet
            icon="ViewRow"
          />
        </ActionGroup>
      </div>
      <div
        class:appGrid={layout === "grid"}
        class:appTable={layout === "table"}
      >
        {#each $apps as app, idx (app._id)}
          <svelte:component
            this={layout === "grid" ? AppCard : AppRow}
            {app}
            {openApp}
            {exportApp}
            {deleteApp}
            last={idx === $apps.length - 1}
          />
        {/each}
      </div>
    </Layout>
  {/if}
  {#if !$apps.length && !creatingApp && loaded}
    <div class="empty-wrapper">
      <Modal inline>
        <ModalContent
          title="Create your first app"
          confirmText="Create app"
          showCancelButton={false}
          showCloseIcon={false}
          onConfirm={initiateAppCreation}
          size="M"
        >
          <div slot="footer">
            <Button on:click={initiateAppImport} secondary>Import app</Button>
          </div>
          <Body size="S">
            The purpose of the Budibase builder is to help you build beautiful,
            powerful applications quickly and easily.
          </Body>
        </ModalContent>
      </Modal>
    </div>
  {/if}
</Page>
<Modal
  bind:this={creationModal}
  padding={false}
  width="600px"
  on:hide={stopAppCreation}
>
  <CreateAppModal {template} />
</Modal>
<ConfirmDialog
  bind:this={deletionModal}
  title="Confirm deletion"
  okText="Delete app"
  onOk={confirmDeleteApp}
>
  Are you sure you want to delete the app <b>{appToDelete?.name}</b>?
</ConfirmDialog>

<style>
  .title,
  .filter {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .select {
    width: 120px;
  }

  .appGrid {
    display: grid;
    grid-gap: 50px;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  }
  .appTable {
    display: grid;
    grid-template-rows: auto;
    grid-template-columns: 1fr 1fr 1fr auto;
    align-items: center;
  }
  .appTable :global(> div) {
    height: 70px;
    display: grid;
    align-items: center;
    gap: var(--spacing-xl);
    grid-template-columns: auto 1fr;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    padding: 0 var(--spacing-s);
  }
  .appTable :global(> div:not(.last)) {
    border-bottom: var(--border-light);
  }

  .empty-wrapper {
    flex: 1 1 auto;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
</style>
