<template>
  <PlaceholderPage
    title="Defect Entry"
    description="Input defect category per model untuk kebutuhan detail model, defect total match, dan worst defect."
    icon="i-lucide-circle-alert"
    :actions="[
      { label: 'Defect Categories', icon: 'i-lucide-bug', to: '/references/defect-categories' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
