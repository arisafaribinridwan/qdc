<script setup lang="ts">
import type { CommandPaletteGroup, CommandPaletteItem, NavigationMenuItem } from '@nuxt/ui'

const route = useRoute()
const toast = useToast()
const open = ref(false)

const closeMobileNav = () => {
  open.value = false
}

const navigation = [
  [
    {
      label: 'Dashboard',
      icon: 'i-lucide-layout-dashboard',
      to: '/',
      exact: true,
      onSelect: closeMobileNav
    },
    {
      label: 'Report Months',
      icon: 'i-lucide-calendar-days',
      to: '/report-months',
      onSelect: closeMobileNav
    },
    {
      label: 'Models',
      icon: 'i-lucide-boxes',
      to: '/models',
      onSelect: closeMobileNav
    }
  ],
  [
    {
      label: 'References',
      icon: 'i-lucide-library',
      defaultOpen: route.path.startsWith('/references'),
      type: 'trigger',
      children: [
        {
          label: 'Defect Categories',
          icon: 'i-lucide-bug',
          to: '/references/defect-categories',
          onSelect: closeMobileNav
        },
        {
          label: 'Non-defect Categories',
          icon: 'i-lucide-shield-check',
          to: '/references/nondefect-categories',
          onSelect: closeMobileNav
        },
        {
          label: 'Grouping Rules',
          icon: 'i-lucide-git-branch',
          to: '/references/grouping-rules',
          onSelect: closeMobileNav
        }
      ]
    },
    {
      label: 'Entry',
      icon: 'i-lucide-clipboard-pen-line',
      defaultOpen: route.path.startsWith('/entry'),
      type: 'trigger',
      children: [
        {
          label: 'Summary',
          icon: 'i-lucide-table-properties',
          to: '/entry/summary',
          onSelect: closeMobileNav
        },
        {
          label: 'Defect',
          icon: 'i-lucide-circle-alert',
          to: '/entry/defect',
          onSelect: closeMobileNav
        },
        {
          label: 'Non-defect',
          icon: 'i-lucide-circle-check',
          to: '/entry/nondefect',
          onSelect: closeMobileNav
        },
        {
          label: 'Repair Action',
          icon: 'i-lucide-wrench',
          to: '/entry/repair-action',
          onSelect: closeMobileNav
        },
        {
          label: 'F-COST',
          icon: 'i-lucide-circle-dollar-sign',
          to: '/entry/fcost',
          onSelect: closeMobileNav
        }
      ]
    },
    {
      label: 'Validation',
      icon: 'i-lucide-badge-check',
      to: '/validation',
      onSelect: closeMobileNav
    },
    {
      label: 'Reports',
      icon: 'i-lucide-file-chart-column',
      defaultOpen: route.path.startsWith('/reports'),
      type: 'trigger',
      children: [
        {
          label: 'Preview',
          icon: 'i-lucide-eye',
          to: '/reports/preview',
          onSelect: closeMobileNav
        },
        {
          label: 'Export',
          icon: 'i-lucide-download',
          to: '/reports/export',
          onSelect: closeMobileNav
        }
      ]
    }
  ],
  [
    {
      label: 'Settings',
      icon: 'i-lucide-settings',
      defaultOpen: route.path.startsWith('/settings'),
      type: 'trigger',
      children: [
        {
          label: 'Backup',
          icon: 'i-lucide-database-backup',
          to: '/settings/backup',
          onSelect: closeMobileNav
        }
      ]
    }
  ]
] satisfies NavigationMenuItem[][]

function toSearchItems(items: NavigationMenuItem[]): CommandPaletteItem[] {
  return items.flatMap((item) => {
    if (item.children?.length) {
      return item.children.map(child => ({
        label: child.label,
        icon: child.icon,
        to: child.to,
        onSelect: child.onSelect
      }))
    }

    return [{
      label: item.label,
      icon: item.icon,
      to: item.to,
      exact: item.exact,
      onSelect: item.onSelect
    }]
  })
}

const groups = computed<CommandPaletteGroup[]>(() => [{
  id: 'navigation',
  label: 'QRCC workflow',
  items: navigation.flatMap(toSearchItems)
}])

onMounted(() => {
  const cookie = useCookie('qdc-shell-phase')

  if (cookie.value === 'phase-1') {
    return
  }

  toast.add({
    title: 'QRCC Data Center shell siap',
    description: 'Phase 1 menyiapkan identity, navigasi, dan placeholder workflow tanpa menyentuh backend.',
    color: 'primary',
    icon: 'i-lucide-layout-dashboard',
    duration: 6000,
    actions: [{
      label: 'OK',
      color: 'neutral',
      variant: 'outline',
      onClick: () => {
        cookie.value = 'phase-1'
      }
    }]
  })
})
</script>

<template>
  <UDashboardGroup unit="rem" storage="cookie" storage-key="qdc-dashboard">
    <UDashboardSidebar
      id="qdc-sidebar"
      v-model:open="open"
      collapsible
      resizable
      class="bg-elevated/25"
      :ui="{ footer: 'lg:border-t lg:border-default' }"
    >
      <template #header="{ collapsed }">
        <TeamsMenu :collapsed="collapsed" />
      </template>

      <template #default="{ collapsed }">
        <UNavigationMenu
          v-for="(items, index) in navigation"
          :key="index"
          :collapsed="collapsed"
          :items="items"
          orientation="vertical"
          tooltip
          popover
          :class="index === navigation.length - 1 ? 'mt-auto' : undefined"
        />
      </template>

      <template #footer="{ collapsed }">
        <UserMenu :collapsed="collapsed" />
      </template>
    </UDashboardSidebar>

    <UDashboardSearch :groups="groups" />

    <slot />

    <NotificationsSlideover />
  </UDashboardGroup>
</template>
