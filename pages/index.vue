<script setup>
const config = useRuntimeConfig()

const { data, error, pending } = await useFetch(
  `${config.public.apiBase}/Regions`
)
</script>

<template>
  <div>
    <h1 class="text-xl font-bold">Regions</h1>

    <div v-if="pending">Loading...</div>
    <div v-else-if="error">Error loading data</div>

    <ul v-else>
      <li v-for="region in data" :key="region.id">
        {{ region.name }}
      </li>
    </ul>
  </div>
</template>