<template>
  <PlaceholderPage
    title="Report Preview"
    description="Preview report FQMS/F-COST yang nantinya menjadi sumber layout PDF dan pemeriksaan sebelum export."
    icon="i-lucide-eye"
    :actions="[
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' },
      { label: 'Export', icon: 'i-lucide-download', to: '/reports/export' }
    ]"
  />
</template>
