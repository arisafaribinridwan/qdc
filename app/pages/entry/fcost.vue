<template>
  <PlaceholderPage
    title="F-COST Entry"
    description="Input F-COST bulanan, item breakdown Part/Labor/Trip, dan kategori part untuk perhitungan rasio report."
    icon="i-lucide-circle-dollar-sign"
    :actions="[
      { label: 'Report Months', icon: 'i-lucide-calendar-days', to: '/report-months' },
      { label: 'Validation', icon: 'i-lucide-badge-check', to: '/validation' }
    ]"
  />
</template>
