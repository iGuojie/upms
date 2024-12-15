<template>
  <v-breadcrumbs :items="breadcrumbs" class="px-4">
    <template v-slot:divider>
      <v-icon icon="mdi-chevron-right"></v-icon>
    </template>
  </v-breadcrumbs>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const breadcrumbs = ref([])

const breadcrumbMap = {
  home: '首页',
  users: '用户管理',
  settings: '系统设置'
}

const updateBreadcrumbs = () => {
  breadcrumbs.value = [
    { title: '首页', disabled: false, href: '/' },
    ...route.path.split('/')
      .filter(Boolean)
      .map((path, index, arr) => ({
        title: breadcrumbMap[path] || path.charAt(0).toUpperCase() + path.slice(1),
        disabled: index === arr.length - 1,
        href: '/' + arr.slice(0, index + 1).join('/')
      }))
  ]
}

watch(() => route.path, updateBreadcrumbs, { immediate: true })
</script>
