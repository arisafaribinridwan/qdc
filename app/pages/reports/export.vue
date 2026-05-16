<template>
  <PlaceholderPage
    title="Report Export"
    description="Export PDF/Excel dan history export untuk output meeting QRCC setelah preview dan validasi siap."
    icon="i-lucide-download"
    :actions="[
      { label: 'Report Preview', icon: 'i-lucide-eye', to: '/reports/preview' },
      { label: 'Backup', icon: 'i-lucide-database-backup', to: '/settings/backup' }
    ]"
  />
</template>
