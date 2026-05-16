<script setup lang="ts">
import type { DropdownMenuItem } from '@nuxt/ui'

defineProps<{
  collapsed?: boolean
}>()

const colorMode = useColorMode()

const user = {
  label: 'QRCC User',
  avatar: {
    alt: 'QRCC User',
    icon: 'i-lucide-user'
  }
}

const items = computed<DropdownMenuItem[][]>(() => [
  [{ type: 'label', label: 'QRCC Data Center' }],
  [
    {
      label: 'Backup settings',
      icon: 'i-lucide-database-backup',
      to: '/settings/backup'
    },
    {
      label: colorMode.value === 'dark' ? 'Light mode' : 'Dark mode',
      icon: colorMode.value === 'dark' ? 'i-lucide-sun' : 'i-lucide-moon',
      onSelect: () => {
        colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark'
      }
    }
  ]
])
</script>

<template>
  <UDropdownMenu
    :items="items"
    :content="{ align: 'center', collisionPadding: 12 }"
    :ui="{ content: collapsed ? 'w-48' : 'w-(--reka-dropdown-menu-trigger-width)' }"
  >
    <UButton
      v-bind="user"
      :label="collapsed ? undefined : user.label"
      :trailing-icon="collapsed ? undefined : 'i-lucide-chevrons-up-down'"
      color="neutral"
      variant="ghost"
      block
      :square="collapsed"
      class="data-[state=open]:bg-elevated"
      :ui="{ trailingIcon: 'text-dimmed' }"
    />
  </UDropdownMenu>
</template>
