<template>
  <PlaceholderPage
    title="Report Months"
    description="Daftar bulan laporan, status aktif, dan kesiapan data bulanan untuk workflow QRCC."
    icon="i-lucide-calendar-days"
    :actions="[
      { label: 'Entry Summary', icon: 'i-lucide-table-properties', to: '/entry/summary' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
