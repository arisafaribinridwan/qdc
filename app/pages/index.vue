<script setup lang="ts">
const { isNotificationsSlideoverOpen } = useDashboard()

const statusCards = [
  {
    title: 'Bulan aktif',
    value: 'Belum dipilih',
    description: 'Report month selector akan terhubung setelah data model tersedia.',
    icon: 'i-lucide-calendar-days',
    color: 'primary' as const
  },
  {
    title: 'Status validasi',
    value: 'Menunggu data',
    description: 'Rule OK/CHECK akan tampil setelah validation engine dibuat.',
    icon: 'i-lucide-badge-check',
    color: 'warning' as const
  },
  {
    title: 'Langkah berikutnya',
    value: 'Lengkapi master data',
    description: 'Mulai dari report months, models, dan reference categories.',
    icon: 'i-lucide-list-checks',
    color: 'neutral' as const
  }
]

const shortcuts = [
  {
    label: 'Report Months',
    description: 'Kelola periode laporan bulanan.',
    icon: 'i-lucide-calendar-days',
    to: '/report-months'
  },
  {
    label: 'Entry Summary',
    description: 'Input monthly model summary.',
    icon: 'i-lucide-table-properties',
    to: '/entry/summary'
  },
  {
    label: 'Validation',
    description: 'Cek kelengkapan dan konsistensi data.',
    icon: 'i-lucide-badge-check',
    to: '/validation'
  },
  {
    label: 'Report Preview',
    description: 'Preview output FQMS/F-COST sebelum export.',
    icon: 'i-lucide-eye',
    to: '/reports/preview'
  }
]
</script>

<template>
  <UDashboardPanel id="dashboard">
    <template #header>
      <UDashboardNavbar title="Dashboard" :ui="{ right: 'gap-3' }">
        <template #leading>
          <UDashboardSidebarCollapse />
        </template>

        <template #right>
          <UTooltip text="Workflow notes" :shortcuts="['N']">
            <UButton
              color="neutral"
              variant="ghost"
              square
              @click="isNotificationsSlideoverOpen = true"
            >
              <UChip color="primary" inset>
                <UIcon name="i-lucide-bell" class="size-5 shrink-0" />
              </UChip>
            </UButton>
          </UTooltip>

          <UButton
            to="/reports/preview"
            label="Generate preview"
            icon="i-lucide-file-chart-column"
          />
        </template>
      </UDashboardNavbar>
    </template>

    <template #body>
      <div class="space-y-8 p-6">
        <section class="space-y-3">
          <UBadge label="Phase 1 · UI Shell" color="primary" variant="subtle" />
          <div class="max-w-3xl space-y-2">
            <h1 class="text-3xl font-semibold tracking-tight text-highlighted">
              QRCC Data Center
            </h1>
            <p class="text-muted">
              Shell dashboard internal untuk workflow FQMS, F-COST, data bulanan, validasi, preview report, dan export. Phase ini hanya menyiapkan navigasi dan placeholder UI tanpa koneksi backend.
            </p>
          </div>
        </section>

        <div class="grid gap-4 lg:grid-cols-3">
          <UCard v-for="card in statusCards" :key="card.title">
            <div class="flex items-start justify-between gap-4">
              <div class="space-y-1">
                <p class="text-sm text-muted">
                  {{ card.title }}
                </p>
                <p class="text-xl font-semibold text-highlighted">
                  {{ card.value }}
                </p>
                <p class="text-sm text-muted">
                  {{ card.description }}
                </p>
              </div>
              <UIcon :name="card.icon" class="size-6 text-primary" />
            </div>
          </UCard>
        </div>

        <section class="space-y-4">
          <div>
            <h2 class="text-lg font-semibold text-highlighted">
              Shortcut workflow
            </h2>
            <p class="text-sm text-muted">
              Placeholder navigasi cepat sesuai PRD Section 9.
            </p>
          </div>

          <div class="grid gap-4 md:grid-cols-2 xl:grid-cols-4">
            <UPageCard
              v-for="shortcut in shortcuts"
              :key="shortcut.to"
              :title="shortcut.label"
              :description="shortcut.description"
              :icon="shortcut.icon"
              :to="shortcut.to"
              variant="subtle"
            />
          </div>
        </section>

        <UAlert
          title="Prinsip dashboard"
          description="Dashboard harus langsung menjawab: bulan apa yang aktif, data apa yang belum lengkap, dan langkah berikutnya apa. Data aktual akan dihubungkan pada phase backend dan data-entry."
          icon="i-lucide-info"
          color="neutral"
          variant="soft"
        />
      </div>
    </template>
  </UDashboardPanel>
</template>
