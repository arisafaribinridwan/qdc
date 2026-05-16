<template>
  <PlaceholderPage
    title="Entry Summary"
    description="Input monthly model summary sebagai sumber utama sales qty, total defect, dan total non-defect."
    icon="i-lucide-table-properties"
    :actions="[
      { label: 'Defect Entry', icon: 'i-lucide-circle-alert', to: '/entry/defect' },
      { label: 'Non-defect Entry', icon: 'i-lucide-circle-check', to: '/entry/nondefect' }
    ]"
  />
</template>
