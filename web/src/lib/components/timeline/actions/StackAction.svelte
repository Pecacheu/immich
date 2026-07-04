<script lang="ts">
  import MenuOption from '$lib/components/shared-components/context-menu/MenuOption.svelte';
  import { assetMultiSelectManager } from '$lib/managers/asset-multi-select-manager.svelte';
  import { updateStackedAssetInTimeline, updateUnstackedAssetInTimeline } from '$lib/utils/actions';
  import { TimelineManager } from '$lib/managers/timeline-manager/timeline-manager.svelte';
  import { deleteStack, stackAssets } from '$lib/utils/asset-utils';
  import { toTimelineAsset } from '$lib/utils/timeline-util';
  import { mdiImageMultipleOutline, mdiImageOffOutline } from '@mdi/js';
  import { t } from 'svelte-i18n';

  interface Props {
    timelineManager?: TimelineManager;
  }

  let { timelineManager }: Props = $props();

  let isAssetStackSelected = $derived(
    assetMultiSelectManager.assets.length === 1 && !!assetMultiSelectManager.assets[0].stack,
  );

  const handleStack = async () => {
    const result = await stackAssets(assetMultiSelectManager.ownedAssets);
    if (timelineManager) updateStackedAssetInTimeline(timelineManager, result);
    assetMultiSelectManager.clear();
  };

  const handleUnstack = async () => {
    const selectedAssets = assetMultiSelectManager.ownedAssets;
    if (selectedAssets.length !== 1) {
      return;
    }
    const { stack } = selectedAssets[0];
    if (!stack) {
      return;
    }
    const unstackedAssets = await deleteStack([stack.id]);
    if (unstackedAssets && timelineManager) {
      updateUnstackedAssetInTimeline(
        timelineManager,
        unstackedAssets.map((a) => toTimelineAsset(a)),
      );
    }
    assetMultiSelectManager.clear();
  };
</script>

{#if assetMultiSelectManager.assets.length > 1 || isAssetStackSelected}
  {#if isAssetStackSelected}
    <MenuOption text={$t('unstack')} icon={mdiImageOffOutline} onClick={handleUnstack} />
  {:else}
    <MenuOption text={$t('stack')} icon={mdiImageMultipleOutline} onClick={handleStack} />
  {/if}
{/if}
