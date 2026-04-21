<template>
  <div class="min-h-screen bg-gray-100 p-8">
    <h1 class="text-3xl font-bold text-gray-800 mb-8">Region Selector</h1>

    <div v-if="pending" class="text-gray-500">Loading regions...</div>

    <div v-else-if="error" class="text-red-500">
      Failed to load regions: {{ error }}
    </div>

    <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6">
      <div
        v-for="region in regions"
        :key="region.id"
        class="bg-white rounded-2xl shadow-md p-6 cursor-pointer hover:shadow-lg hover:scale-105 transition-transform duration-200"
        :class="{ 'ring-2 ring-blue-500': selected && selected.id === region.id }"
        @click="selected = region"
      >
        <div class="text-sm font-semibold text-blue-500 uppercase tracking-wide mb-1">
          {{ region.code }}
        </div>
        <div class="text-xl font-bold text-gray-800">{{ region.name }}</div>
        <div class="text-xs text-gray-400 mt-2 truncate">{{ region.regionImageUrl }}</div>
      </div>
    </div>

    <div v-if="selected" class="mt-8 bg-white rounded-2xl shadow p-6 max-w-sm">
      <h2 class="text-lg font-bold text-gray-700 mb-1">Selected Region</h2>
      <p class="text-blue-600 font-semibold">{{ selected.name }}</p>
      <p class="text-gray-400 text-sm">Code: {{ selected.code }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      regions: [],
      selected: null,
      pending: true,
      error: null
    }
  },
  async mounted() {
    try {
      const data = await fetch('https://localhost:7077/api/Regions')
      this.regions = await data.json()
    } catch (err) {
      this.error = err.message || 'Unknown error'
    } finally {
      this.pending = false
    }
  }
}
</script>
