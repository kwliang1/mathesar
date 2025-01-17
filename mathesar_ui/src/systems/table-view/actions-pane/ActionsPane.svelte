<script lang="ts">
  import { Button, Icon } from '@mathesar-component-library';
  import type { TableEntry } from '@mathesar/api/types/tables';
  import type { Database, SchemaEntry } from '@mathesar/AppTypes';
  import ModificationStatus from '@mathesar/components/ModificationStatus.svelte';
  import TableNameAndDescription from '@mathesar/components/TableNameAndDescription.svelte';
  import { iconInspector } from '@mathesar/icons';
  import { getTabularDataStoreFromContext } from '@mathesar/stores/table-data';
  import { constructDataExplorerUrlToSummarizeFromGroup } from '@mathesar/systems/data-explorer';
  import FilterDropdown from './record-operations/filter/FilterDropdown.svelte';
  import GroupDropdown from './record-operations/group/GroupDropdown.svelte';
  import SortDropdown from './record-operations/sort/SortDropdown.svelte';
  import SummarizationLink from './SummarizationLink.svelte';

  export let database: Database;
  export let schema: SchemaEntry;
  export let table: Pick<TableEntry, 'name' | 'description'>;

  const tabularData = getTabularDataStoreFromContext();

  $: ({ id, columnsDataStore, meta, isLoading, display } = $tabularData);
  $: ({ columns } = columnsDataStore);
  $: ({ filtering, sorting, grouping, sheetState } = meta);
  $: ({ isTableInspectorVisible } = display);
  $: summarizationUrl = constructDataExplorerUrlToSummarizeFromGroup(
    database.name,
    schema.id,
    {
      baseTable: {
        id,
        name: table.name,
      },
      columns: $columns,
      terseGrouping: $grouping.terse(),
    },
  );

  function toggleTableInspector() {
    isTableInspectorVisible.set(!$isTableInspectorVisible);
  }
</script>

<div class="actions-pane">
  <div class="heading">
    <TableNameAndDescription {table} />
  </div>

  <div class="actions">
    <div class="quick-access">
      <FilterDropdown {filtering} />
      <SortDropdown {sorting} columns={$columns} />
      <GroupDropdown {grouping} />
    </div>

    <ModificationStatus requestState={$sheetState} />

    <div class="aux-actions">
      {#if summarizationUrl}
        <SummarizationLink {summarizationUrl} />
      {/if}
      <Button
        appearance="secondary"
        size="medium"
        disabled={$isLoading}
        on:click={toggleTableInspector}
        active={$isTableInspectorVisible}
      >
        <Icon {...iconInspector} />
        <span>Inspector</span>
      </Button>
    </div>
  </div>
</div>

<style lang="scss">
  .actions-pane {
    --badge-font-size: var(--text-size-small);
    border-bottom: 1px solid var(--slate-300);
    background-color: var(--color-white);
    position: relative;
    display: flex;
    align-items: center;
  }
  .heading {
    /**
    * restricting the max-width 
    * so that long descriptions does not take all the available space
    */
    max-width: 20%;
    padding: 1rem;
    font-size: var(--text-size-large);
  }
  .actions {
    flex: 1;
    border-left: 1px solid var(--slate-300);
    display: flex;
    padding: 1rem;
    padding: 1rem;
    flex-direction: row;
    align-items: center;
  }
  .quick-access {
    display: flex;
    flex-direction: row;
    margin-right: 0.6em;

    > :global(* + *) {
      margin-left: 0.5rem;
    }
  }

  .aux-actions {
    display: flex;
    flex-direction: row;
    align-items: center;
    margin-left: auto;

    > :global(* + *) {
      margin-left: 1rem;
    }
  }
</style>
