<script setup lang="ts">
const props = defineProps<{
  title: string
  description: string
  icon: string
  phase?: string
  actions?: Array<{ label: string, icon: string, to: string }>
}>()

const phaseLabel = computed(() => props.phase ?? 'Frontend placeholder')
const { isNotificationsSlideoverOpen } = useDashboard()
</script>

<template>
  <UDashboardPanel :id="title.toLowerCase().replaceAll(' ', '-')">
    <template #header>
      <UDashboardNavbar :title="title">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>

        <template #right>
          <UButton
            icon="i-lucide-bell"
            color="neutral"
            variant="ghost"
            square
            @click="isNotificationsSlideoverOpen = true"
          />
        </template>
      </UDashboardNavbar>
    </template>

    <template #body>
      <div class="mx-auto flex min-h-[calc(100vh-10rem)] w-full max-w-5xl items-center justify-center p-6">
        <UCard class="w-full max-w-2xl">
          <div class="flex flex-col gap-6 text-center sm:flex-row sm:text-left">
            <div class="mx-auto flex size-14 shrink-0 items-center justify-center rounded-2xl bg-primary/10 text-primary sm:mx-0">
              <UIcon :name="icon" class="size-7" />
            </div>

            <div class="min-w-0 flex-1 space-y-4">
              <div class="space-y-2">
                <UBadge :label="phaseLabel" color="primary" variant="subtle" />
                <h1 class="text-2xl font-semibold tracking-tight text-highlighted">
                  {{ title }}
                </h1>
                <p class="text-muted">
                  {{ description }}
                </p>
              </div>

              <UAlert
                title="Belum terhubung ke backend"
                description="Halaman ini sengaja disiapkan sebagai shell UI Phase 1. Form, tabel, schema, dan API akan masuk pada phase implementasi berikutnya."
                icon="i-lucide-info"
                color="neutral"
                variant="soft"
              />

              <div v-if="actions?.length" class="flex flex-wrap gap-2">
                <UButton
                  v-for="action in actions"
                  :key="action.to"
                  :to="action.to"
                  :label="action.label"
                  :icon="action.icon"
                  color="neutral"
                  variant="outline"
                />
              </div>
            </div>
          </div>
        </UCard>
      </div>
    </template>
  </UDashboardPanel>
</template>
