<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-100 flex items-center justify-center p-6">

    <div class="w-full max-w-2xl">

      <!-- HEADER -->
      <div class="text-center mb-8">
        <p class="text-sm font-semibold tracking-widest text-indigo-600 uppercase">
          NZ Walks Admin
        </p>
        <h1 class="text-4xl font-bold text-gray-800 mt-2">
          Add a <span class="text-indigo-600">Region</span>
        </h1>
      </div>

      <!-- CARD -->
      <div class="bg-white/80 backdrop-blur-xl shadow-xl rounded-2xl p-8 border border-gray-100">

        <!-- FORM -->
        <div v-if="!success">

          <!-- ERROR BANNER -->
          <div
            v-if="errorBanner"
            class="mb-5 p-3 rounded-lg bg-red-50 border border-red-200 text-red-600 text-sm"
          >
            {{ errorBanner }}
          </div>

          <!-- CODE -->
          <div class="mb-5">
            <label class="text-sm font-medium text-gray-700">Region Code</label>
            <input
              v-model="form.code"
              placeholder="e.g. CLMBN"
              maxlength="10"
              class="mt-1 w-full px-4 py-3 rounded-lg border focus:ring-2 focus:ring-indigo-400 focus:border-indigo-500 outline-none transition"
              :class="errors.code ? 'border-red-400' : 'border-gray-200'"
            />
            <p v-if="errors.code" class="text-xs text-red-500 mt-1">
              {{ errors.code }}
            </p>
            <p class="text-xs text-gray-400 mt-1">
              Short uppercase identifier for the region
            </p>
          </div>

          <!-- NAME -->
          <div class="mb-5">
            <label class="text-sm font-medium text-gray-700">Region Name</label>
            <input
              v-model="form.name"
              placeholder="e.g. Colombo North"
              class="mt-1 w-full px-4 py-3 rounded-lg border focus:ring-2 focus:ring-indigo-400 focus:border-indigo-500 outline-none transition"
              :class="errors.name ? 'border-red-400' : 'border-gray-200'"
            />
            <p v-if="errors.name" class="text-xs text-red-500 mt-1">
              {{ errors.name }}
            </p>
          </div>

          <!-- IMAGE -->
          <div class="mb-6">
            <label class="text-sm font-medium text-gray-700">Region Image URL</label>
            <input
              v-model="form.regionImageUrl"
              placeholder="e.g. colombo-north.jpg"
              class="mt-1 w-full px-4 py-3 rounded-lg border border-gray-200 focus:ring-2 focus:ring-indigo-400 focus:border-indigo-500 outline-none transition"
            />
            <p class="text-xs text-gray-400 mt-1">
              Optional — leave blank if not available
            </p>
          </div>

          <!-- BUTTON -->
          <button
            @click="submitForm"
            :disabled="loading"
            class="w-full flex items-center justify-center gap-2 py-3 rounded-lg text-white font-semibold transition
            bg-indigo-600 hover:bg-indigo-700 active:scale-[0.98] disabled:opacity-60"
          >
            <svg v-if="loading" class="animate-spin h-5 w-5 border-2 border-white border-t-transparent rounded-full"></svg>
            {{ loading ? 'Adding...' : 'Add Region' }}
          </button>
        </div>

        <!-- SUCCESS -->
        <div v-else class="text-center py-6">

          <div class="mx-auto w-16 h-16 flex items-center justify-center rounded-full bg-green-100 text-green-600 text-2xl font-bold">
            ✓
          </div>

          <h2 class="text-2xl font-bold mt-4 text-gray-800">
            Region Added!
          </h2>

          <p class="text-gray-500 mt-2">
            The region has been successfully saved to the database.
          </p>

          <div class="mt-4 inline-block px-4 py-2 bg-gray-100 rounded-lg text-sm font-medium text-gray-700">
            {{ success.code }} — {{ success.name }}
          </div>

          <button
            @click="resetForm"
            class="mt-6 px-5 py-2 rounded-lg bg-indigo-600 text-white hover:bg-indigo-700 transition"
          >
            + Add Another Region
          </button>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        code: '',
        name: '',
        regionImageUrl: ''
      },
      errors: {
        code: '',
        name: ''
      },
      loading: false,
      errorBanner: '',
      success: null,
      config: useRuntimeConfig()
    }
  },

  computed: {
    API_BASE() {
      return this.config.public.apiBase
    }
  },

  methods: {
    validate() {
      this.errors.code = this.form.code.trim() ? '' : 'Region code is required.'
      this.errors.name = this.form.name.trim() ? '' : 'Region name is required.'
      return !this.errors.code && !this.errors.name
    },

    async submitForm() {
      if (!this.validate()) return

      this.loading = true
      this.errorBanner = ''

      try {
        const data = await $fetch(`${this.API_BASE}/Regions`, {
          method: 'POST',
          body: {
            code: this.form.code.trim(),
            name: this.form.name.trim(),
            regionImageUrl: this.form.regionImageUrl.trim() || null
          }
        })

        this.success = data
      } catch (err) {
        this.errorBanner =
          err?.data?.message || err.message || 'Something went wrong.'
      } finally {
        this.loading = false
      }
    },

    resetForm() {
      this.form.code = ''
      this.form.name = ''
      this.form.regionImageUrl = ''
      this.success = null
      this.errorBanner = ''
    }
  }
}
</script>