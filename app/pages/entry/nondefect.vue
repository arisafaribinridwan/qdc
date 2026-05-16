<template>
  <PlaceholderPage
    title="Non-defect Entry"
    description="Input non-defect category per model untuk menjaga konsistensi total non-defect pada report bulanan."
    icon="i-lucide-circle-check"
    :actions="[
      { label: 'Non-defect Categories', icon: 'i-lucide-shield-check', to: '/references/nondefect-categories' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
