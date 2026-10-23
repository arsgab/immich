<script lang="ts">
  import { Icon } from '@immich/ui';
  import { mdiChevronRight, mdiChevronLeft } from '@mdi/js';
  import { t } from 'svelte-i18n';
  import { shortcuts } from '$lib/actions/shortcut';
  import { languageManager } from '$lib/managers/language-manager.svelte';
  import NavigationArea from '../NavigationArea.svelte';

  interface Props {
    onNextAsset: () => void;
  }

  let { onNextAsset }: Props = $props();
</script>

<svelte:document
  use:shortcuts={[
    { shortcut: { key: 'ArrowRight' }, onShortcut: onNextAsset },
    { shortcut: { key: 'd' }, onShortcut: onNextAsset },
  ]}
/>

<NavigationArea onClick={onNextAsset} label={$t('view_next_asset')}>
  <Icon icon={languageManager.rtl ? mdiChevronLeft : mdiChevronRight} size="36" aria-hidden />
</NavigationArea>
