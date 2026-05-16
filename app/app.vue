<script setup lang="ts">
const colorMode = useColorMode()

const color = computed(() => colorMode.value === 'dark' ? '#0f172a' : '#ffffff')

useHead({
  meta: [
    { charset: 'utf-8' },
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    { key: 'theme-color', name: 'theme-color', content: color }
  ],
  link: [{ rel: 'icon', href: '/favicon.png' }],
  htmlAttrs: {
    lang: 'id'
  }
})

const title = 'QRCC Data Center'
const description = 'Aplikasi dashboard internal QRCC untuk mengelola workflow FQMS, F-COST, PPM, Sales, Quality Issue, validasi data bulanan, dan report generation.'

useSeoMeta({
  title,
  description,
  ogTitle: title,
  ogDescription: description,
  ogImage: '/favicon.png',
  twitterImage: '/favicon.png',
  twitterCard: 'summary'
})
</script>

<template>
  <UApp>
    <NuxtLoadingIndicator />

    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </UApp>
</template>
