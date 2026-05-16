<template>
  <PlaceholderPage
    title="Repair Action"
    description="Input repair action per model sebagai konteks tindak lanjut quality issue di workflow QRCC."
    icon="i-lucide-wrench"
    :actions="[
      { label: 'Defect Entry', icon: 'i-lucide-circle-alert', to: '/entry/defect' },
      { label: 'Report Preview', icon: 'i-lucide-eye', to: '/reports/preview' }
    ]"
  />
</template>
