<template>
  <PlaceholderPage
    title="Models"
    description="Master model untuk mengatur active status, report include, dan dasar grouping FQMS/F-COST."
    icon="i-lucide-boxes"
    :actions="[
      { label: 'Grouping Rules', icon: 'i-lucide-git-branch', to: '/references/grouping-rules' },
      { label: 'Entry Summary', icon: 'i-lucide-table-properties', to: '/entry/summary' }
    ]"
  />
</template>
