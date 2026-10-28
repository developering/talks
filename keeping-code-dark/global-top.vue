<script setup lang="ts">
import { computed, watchEffect } from 'vue'
import { useNav } from '@slidev/client'
import { isDark } from '@slidev/client/logic/dark.ts'

const { slides, currentSlideNo, clicks } = useNav()

const flip = computed(() => {
  const route = slides.value.find(s => s.meta?.slide?.frontmatter?.lightsOnAtClick !== undefined)
  return route && { no: route.no, click: route.meta.slide.frontmatter.lightsOnAtClick as number }
})

watchEffect(() => {
  const f = flip.value
  if (!f) return
  const lightsOn = currentSlideNo.value > f.no
    || (currentSlideNo.value === f.no && clicks.value >= f.click)
  isDark.value = !lightsOn
})
</script>

<template></template>
