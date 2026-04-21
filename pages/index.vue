<template>
  <div class="min-h-screen bg-gray-100 p-8">
    <h1 class="text-2xl font-bold text-gray-800 mb-6">Regions</h1>

    <div v-if="pending" class="text-gray-400 text-sm">Loading regions...</div>
    <div v-else-if="error" class="text-red-500 text-sm">Failed to load: {{ error }}</div>

    <div v-else class="flex h-[560px] border border-gray-200 rounded-2xl overflow-hidden bg-white shadow-sm">

      <!-- LEFT: List panel -->
      <div class="w-52 border-r border-gray-200 overflow-y-auto flex-shrink-0 flex flex-col">

        <!-- Search Box -->
        <div class="px-3 py-2 border-b border-gray-100 sticky top-0 bg-white z-10">
          <div class="flex items-center gap-2 bg-gray-100 rounded-lg px-3 py-1.5">
            <svg class="w-3.5 h-3.5 text-gray-400 flex-shrink-0" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-4.35-4.35M17 11A6 6 0 1 1 5 11a6 6 0 0 1 12 0z"/>
            </svg>
            <input
              v-model="searchQuery"
              @input="onSearch"
              type="text"
              placeholder="Search regions..."
              class="bg-transparent text-xs text-gray-700 placeholder-gray-400 outline-none w-full"
            />
            <button v-if="searchQuery" @click="clearSearch" class="text-gray-400 hover:text-gray-600">
              <svg class="w-3 h-3" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12"/>
              </svg>
            </button>
          </div>
        </div>

        <div class="px-4 py-3 text-xs font-semibold text-gray-400 uppercase tracking-widest border-b border-gray-100 sticky top-[52px] bg-white">
          {{ searchQuery ? `Results (${displayedRegions.length})` : 'All Regions' }}
        </div>

        <!-- Searching indicator -->
        <div v-if="searching" class="px-4 py-3 text-xs text-gray-400">Searching...</div>

        <!-- No results -->
        <div v-else-if="displayedRegions.length === 0 && searchQuery" class="px-4 py-3 text-xs text-gray-400">
          No regions found for "{{ searchQuery }}"
        </div>

        <!-- Region list -->
        <div
          v-for="region in displayedRegions"
          :key="region.id"
          class="flex items-center gap-2 px-4 py-3 cursor-pointer border-b border-gray-100 text-sm transition-colors duration-150"
          :class="selected && selected.id === region.id
            ? 'bg-blue-50 text-blue-600 font-medium'
            : 'text-gray-700 hover:bg-gray-50'"
          @click="selected = region"
        >
          <span
            class="w-1.5 h-1.5 rounded-full flex-shrink-0"
            :class="selected && selected.id === region.id ? 'bg-blue-500' : 'bg-gray-300'"
          />
          {{ region.name }}
        </div>
      </div>

      <!-- RIGHT: Detail panel -->
      <div class="flex-1 flex items-center justify-center p-8 overflow-y-auto">

        <!-- Empty state -->
        <div v-if="!selected" class="text-center text-gray-400">
          <div class="w-12 h-12 rounded-full bg-gray-100 flex items-center justify-center mx-auto mb-3">
            <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round"
                d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-1.447-.894L15 9m0 8V9m0 0L9 7"/>
            </svg>
          </div>
          <p class="text-sm">Select a region to view details</p>
        </div>

        <!-- Detail card -->
        <div v-else class="w-full max-w-sm border border-gray-200 rounded-2xl overflow-hidden">
          <div class="px-5 py-5 border-b border-gray-100">
            <span class="inline-block text-xs font-semibold tracking-widest uppercase bg-blue-50 text-blue-600 px-3 py-1 rounded-full mb-3">
              {{ selected.code }}
            </span>
            <h2 class="text-xl font-bold text-gray-800">{{ selected.name }}</h2>
          </div>
          <div class="px-5 py-4 space-y-4">
            <div>
              <p class="text-xs font-semibold uppercase tracking-widest text-gray-400 mb-1">Region ID</p>
              <p class="text-sm font-mono text-gray-600 break-all">{{ selected.id }}</p>
            </div>
            <div>
              <p class="text-xs font-semibold uppercase tracking-widest text-gray-400 mb-1">Code</p>
              <p class="text-sm text-gray-800">{{ selected.code }}</p>
            </div>
            <div>
              <p class="text-xs font-semibold uppercase tracking-widest text-gray-400 mb-2">Image</p>
              <div class="w-full h-24 rounded-xl bg-gray-100 overflow-hidden flex items-center justify-center text-xs text-gray-400">
                <img
                  v-if="selected.regionImageUrl"
                  :src="selected.regionImageUrl"
                  :alt="selected.name"
                  class="w-full h-full object-cover"
                />
                <span v-else>No image</span>
              </div>
            </div>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      regions: [],
      displayedRegions: [],
      selected: null,
      pending: true,
      searching: false,
      error: null,
      searchQuery: '',
      searchTimeout: null
    }
  },
  async mounted() {
    try {
      const res = await fetch('https://localhost:7077/api/Regions')
      if (!res.ok) throw new Error(`Server error: ${res.status}`)
      this.regions = await res.json()
      this.displayedRegions = this.regions
    } catch (err) {
      this.error = err.message || 'Unknown error'
    } finally {
      this.pending = false
    }
  },
  methods: {
    onSearch() {
      // Clear previous debounce timer
      clearTimeout(this.searchTimeout)

      if (!this.searchQuery.trim()) {
        // If search is cleared, restore full list immediately
        this.displayedRegions = this.regions
        this.selected = null
        return
      }

      // Debounce: wait 400ms after user stops typing before calling API
      this.searching = true
      this.searchTimeout = setTimeout(async () => {
        try {
          const res = await fetch(
            `https://localhost:7077/api/Regions/searchRegion?value=${encodeURIComponent(this.searchQuery.trim())}`
          )
          if (res.status === 404) {
            this.displayedRegions = []
          } else if (!res.ok) {
            throw new Error(`Server error: ${res.status}`)
          } else {
            this.displayedRegions = await res.json()
          }
          this.selected = null
        } catch (err) {
          this.displayedRegions = []
        } finally {
          this.searching = false
        }
      }, 400)
    },

    clearSearch() {
      this.searchQuery = ''
      this.displayedRegions = this.regions
      this.selected = null
      clearTimeout(this.searchTimeout)
    }
  }
}
</script>