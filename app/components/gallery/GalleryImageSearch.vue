<script setup lang="ts">
import { ref } from 'vue'
import { ApiError } from '~/types/api'
import { sha256Hex } from '~/utils/hash'

const { t } = useI18n()
const { api } = useApi()
const { push } = useToasts()
const router = useRouter()
const localePath = useLocalePath()

const fileInput = ref<HTMLInputElement | null>(null)
const searching = ref(false)

function openPicker(): void {
  if (searching.value) return
  fileInput.value?.click()
}

async function onPick(event: Event): Promise<void> {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  input.value = '' // allow re-picking the same file
  if (!file) return
  if (!file.type.startsWith('image/')) {
    push(t('gallery.imageSearchNotImage'), { tone: 'warn' })
    return
  }

  searching.value = true
  try {
    // Same SHA-256 the upload flow computes, so it matches the stored contentHash.
    const hash = await sha256Hex(await file.arrayBuffer())
    const image = await api.getImageByHash(hash)
    await router.push(localePath(`/image/${image.id}`))
  } catch (error: unknown) {
    if (error instanceof ApiError && error.status === 404) {
      push(t('gallery.imageNotFound'), { tone: 'warn' })
    } else {
      push(t('gallery.imageSearchFailed'), { tone: 'error' })
    }
  } finally {
    searching.value = false
  }
}
</script>

<template>
  <LButton
    variant="ghost"
    size="md"
    icon="imageSearch"
    :loading="searching"
    :aria-label="t('gallery.searchByImage')"
    :title="t('gallery.searchByImage')"
    @click="openPicker"
  />
  <input
    ref="fileInput"
    type="file"
    accept="image/*"
    class="gis__input"
    tabindex="-1"
    aria-hidden="true"
    @change="onPick"
  />
</template>

<style scoped>
.gis__input {
  display: none;
}
</style>
