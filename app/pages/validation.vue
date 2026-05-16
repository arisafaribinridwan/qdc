<template>
  <PlaceholderPage
    title="Validation"
    description="Status OK/CHECK untuk kelengkapan master data, consistency report month, defect totals, sales qty, dan F-COST completeness."
    icon="i-lucide-badge-check"
    :actions="[
      { label: 'Entry Summary', icon: 'i-lucide-table-properties', to: '/entry/summary' },
      { label: 'Report Preview', icon: 'i-lucide-eye', to: '/reports/preview' }
    ]"
  />
</template>
