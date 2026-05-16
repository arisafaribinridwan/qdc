<script setup lang="ts">
const { isNotificationsSlideoverOpen } = useDashboard()

const items = [
  {
    title: 'Phase 1 shell aktif',
    description: 'Navigasi domain QRCC sudah diarahkan ke page map FQMS/F-COST.',
    icon: 'i-lucide-layout-dashboard'
  },
  {
    title: 'Backend belum dihubungkan',
    description: 'Halaman saat ini masih placeholder sampai schema dan API masuk phase berikutnya.',
    icon: 'i-lucide-plug-zap'
  }
]
</script>

<template>
  <USlideover
    v-model:open="isNotificationsSlideoverOpen"
    title="Workflow notes"
    description="Catatan sementara untuk shell aplikasi."
  >
    <template #body>
      <div class="space-y-3">
        <UAlert
          v-for="item in items"
          :key="item.title"
          :title="item.title"
          :description="item.description"
          :icon="item.icon"
          color="neutral"
          variant="soft"
        />
      </div>
    </template>
  </USlideover>
</template>
