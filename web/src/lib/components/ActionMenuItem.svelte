<script lang="ts">
  import { type ActionItem } from '@immich/ui';
  import MenuOption from '$lib/components/shared-components/context-menu/MenuOption.svelte';
  import { isEnabled } from '$lib/utils';

  type Props = {
    action: ActionItem;
  };

  const { action }: Props = $props();
  const { title, icon, onAction } = $derived(action);
</script>

{#if icon && isEnabled(action)}
  <MenuOption {icon} text={title} onClick={(event: Event) => onAction({ event, action })} />
{/if}
