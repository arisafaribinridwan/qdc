<template>
  <PlaceholderPage
    title="Backup"
    description="Backup dan restore database untuk menjaga keamanan data workflow bulanan QRCC."
    icon="i-lucide-database-backup"
    :actions="[
      { label: 'Dashboard', icon: 'i-lucide-layout-dashboard', to: '/' },
      { label: 'Export History', icon: 'i-lucide-download', to: '/reports/export' }
    ]"
  />
</template>
