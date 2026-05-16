<script setup lang="ts">
defineProps<{
  collapsed?: boolean
}>()
</script>

<template>
  <div class="flex min-w-0 items-center gap-2 px-1.5 py-1">
    <UAvatar
      src="/favicon.png"
      alt="QRCC Data Center"
      size="sm"
      class="bg-primary/10 ring ring-primary/20"
    />

    <div v-if="!collapsed" class="min-w-0">
      <p class="truncate text-sm font-semibold text-highlighted">
        QRCC Data Center
      </p>
      <p class="truncate text-xs text-muted">
        FQMS · F-COST workflow
      </p>
    </div>
  </div>
</template>
