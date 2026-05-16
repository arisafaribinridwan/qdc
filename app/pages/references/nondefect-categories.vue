<template>
  <PlaceholderPage
    title="Non-defect Categories"
    description="Reference kategori non-defect untuk memastikan input non-defect bulanan memakai standar yang sama."
    icon="i-lucide-shield-check"
    :actions="[
      { label: 'Non-defect Entry', icon: 'i-lucide-circle-check', to: '/entry/nondefect' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
