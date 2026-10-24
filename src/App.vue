<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header class="topbar">
    <RouterLink class="brand" to="/">寻迹重大</RouterLink>
    <nav><RouterLink to="/">首页</RouterLink><RouterLink to="/test">能力测试</RouterLink></nav>
  </header>
  <RouterView />
</template>

<style scoped>
.topbar { height: 56px; display: flex; align-items: center; justify-content: space-between; padding: 0 24px; border-bottom: 1px solid #eee; }
.brand { font-weight: 600; color: #222; }
nav { display: flex; gap: 20px; }
nav a { color: #666; }
nav a.router-link-active { color: #409eff; }
@media (max-width: 480px) { .topbar { padding: 0 16px; } }
</style>
