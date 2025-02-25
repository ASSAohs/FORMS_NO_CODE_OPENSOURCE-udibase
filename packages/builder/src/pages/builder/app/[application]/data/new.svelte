<script>
  import { API } from "api"
  import {
    tables,
    datasources,
    sortedIntegrations as integrations,
  } from "stores/backend"

  import { hasData } from "stores/selectors"
  import { notifications, Body } from "@budibase/bbui"
  import { params, goto } from "@roxi/routify"
  import CreateExternalDatasourceModal from "./_components/CreateExternalDatasourceModal/index.svelte"
  import CreateInternalTableModal from "./_components/CreateInternalTableModal.svelte"
  import DatasourceOption from "./_components/DatasourceOption.svelte"
  import IntegrationIcon from "components/backend/DatasourceNavigator/IntegrationIcon.svelte"
  import CreationPage from "components/common/CreationPage.svelte"
  import ICONS from "components/backend/DatasourceNavigator/icons/index.js"
  import FontAwesomeIcon from "components/common/FontAwesomeIcon.svelte"

  let internalTableModal
  let externalDatasourceModal

  let sampleDataLoading = false
  let externalDatasourceLoading = false

  $: disabled = sampleDataLoading || externalDatasourceLoading

  const createSampleData = async () => {
    sampleDataLoading = true

    try {
      await API.addSampleData($params.application)
      await tables.fetch()
      await datasources.fetch()
      $goto("./table")
    } catch (e) {
      sampleDataLoading = false
      notifications.error("Error creating datasource")
    }
  }
</script>

<CreateInternalTableModal bind:this={internalTableModal} />

<CreateExternalDatasourceModal
  bind:loading={externalDatasourceLoading}
  bind:this={externalDatasourceModal}
/>

<CreationPage
  showClose={hasData($datasources, $tables)}
  onClose={() => $goto("./table")}
  heading="Add new data source"
>
  <div class="subHeading">
    <Body>Get started with our Budibase DB</Body>
    <div
      role="tooltip"
      title="Budibase DB is built with CouchDB"
      class="tooltip"
    >
      <FontAwesomeIcon name="fa-solid fa-circle-info" />
    </div>
  </div>

  <div class="options">
    <DatasourceOption
      on:click={internalTableModal.show}
      title="Create new table"
      description="Non-relational"
      {disabled}
    >
      <svelte:component this={ICONS.BUDIBASE} height="20" width="20" />
    </DatasourceOption>
    <DatasourceOption
      on:click={createSampleData}
      title="Use sample data"
      description="Non-relational"
      disabled={disabled || $datasources.hasDefaultData}
    >
      <svelte:component this={ICONS.BUDIBASE} height="20" width="20" />
    </DatasourceOption>
    <DatasourceOption
      on:click={() => internalTableModal.show({ promptUpload: true })}
      title="Upload data"
      description="Non-relational"
      {disabled}
    >
      <svelte:component this={ICONS.BUDIBASE} height="20" width="20" />
    </DatasourceOption>
  </div>

  <div class="subHeading">
    <Body>Or connect to an external datasource</Body>
  </div>

  <div class="options">
    {#each $integrations as integration}
      <DatasourceOption
        on:click={() => externalDatasourceModal.show(integration)}
        title={integration.friendlyName}
        description={integration.type}
        {disabled}
      >
        <IntegrationIcon
          integrationType={integration.name}
          schema={integration}
        />
      </DatasourceOption>
    {/each}
  </div>
</CreationPage>

<style>
  .subHeading {
    display: flex;
    align-items: center;
    margin-top: 12px;
    margin-bottom: 24px;
  }

  .tooltip {
    margin-left: 6px;
  }

  .options {
    width: 100%;
    display: grid;
    column-gap: 24px;
    row-gap: 24px;
    grid-template-columns: repeat(auto-fit, 235px);
    justify-content: center;
    margin-bottom: 48px;
    max-width: 1050px;
  }
</style>
