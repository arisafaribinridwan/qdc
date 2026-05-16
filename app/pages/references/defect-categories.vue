<template>
  <PlaceholderPage
    title="Defect Categories"
    description="Reference kategori defect agar input defect per model konsisten dan bisa dipakai untuk worst defect report."
    icon="i-lucide-bug"
    :actions="[
      { label: 'Defect Entry', icon: 'i-lucide-circle-alert', to: '/entry/defect' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
