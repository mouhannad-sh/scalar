<script lang="ts" setup>
import { type StatesArray } from '@hocuspocus/provider'
import { type ThemeId, ThemeStyles } from '@scalar/themes'
import { computed, isRef, ref, watch } from 'vue'

import {
  type OpenSwaggerEditorActions,
  type SwaggerEditorProps,
} from '../../types'
import ResetStyles from '../ResetStyles.vue'
import SwaggerEditorHeader from './SwaggerEditorHeader.vue'
import SwaggerEditorInput from './SwaggerEditorInput.vue'
import SwaggerEditorNotification from './SwaggerEditorNotification.vue'
import SwaggerEditorStatusBar from './SwaggerEditorStatusBar.vue'

const props = defineProps<SwaggerEditorProps>()

const emit = defineEmits<{
  (e: 'contentUpdate', value: string): void
  (e: 'import', value: string): void
  (e: 'changeTheme', value: ThemeId): void
}>()

const swaggerEditorHeaderRef = ref<typeof SwaggerEditorHeader | null>(null)

const awarenessStates = ref<StatesArray>([])

const handleContentUpdate = (value: string) => {
  emit('contentUpdate', value)
}

const handleAwarenessUpdate = (states: StatesArray) => {
  awarenessStates.value = states
}

const codeMirrorReference = ref<typeof SwaggerEditorInput | null>(null)

const formattedError = computed(() => {
  // Check whether there‘s an error
  if (props.error === undefined || props.error === null || props.error === '') {
    return ''
  }

  // Work with strings and refs
  const error = isRef(props.error) ? props.error.value : props.error

  // Handle YAMLExceptions
  if (error.startsWith('YAMLException:')) {
    // Trim everything but the first line
    return error.split('\n')[0]
  }

  return error
})

watch(
  () => props.value,
  async () => {
    if (props.value) {
      handleContentUpdate(props.value)
    }
  },
  { immediate: true },
)

const handleOpenSwaggerEditor = (action?: OpenSwaggerEditorActions) => {
  if (action === 'importUrl') {
    swaggerEditorHeaderRef?.value?.importUrlModal?.show()
  } else if (action === 'uploadFile') {
    swaggerEditorHeaderRef?.value?.openFileDialog()
  }
}

defineExpose({
  handleOpenSwaggerEditor,
})
</script>
<template>
  <ThemeStyles :id="theme" />
  <ResetStyles v-slot="{ styles }">
    <div
      class="swagger-editor"
      :class="styles">
      <SwaggerEditorHeader
        ref="swaggerEditorHeaderRef"
        :proxyUrl="proxyUrl"
        @import="handleContentUpdate">
        <template #tab-items><slot name="tab-items" /></template>
      </SwaggerEditorHeader>
      <SwaggerEditorNotification v-if="formattedError">
        {{ formattedError }}
      </SwaggerEditorNotification>
      <SwaggerEditorInput
        ref="codeMirrorReference"
        :hocuspocusConfiguration="hocuspocusConfiguration"
        :value="value"
        @awarenessUpdate="handleAwarenessUpdate"
        @contentUpdate="handleContentUpdate" />
      <SwaggerEditorStatusBar v-if="awarenessStates.length">
        {{ awarenessStates.length }} user{{
          awarenessStates.length === 1 ? '' : 's'
        }}
        online
      </SwaggerEditorStatusBar>
    </div>
  </ResetStyles>
</template>
<style scoped>
.swagger-editor {
  min-width: 0;
  min-height: 0;

  display: flex;
  flex: 1;
  flex-direction: column;
  overflow: auto;
  border-right: 1px solid
    var(--theme-border-color, var(--default-theme-border-color));
  font-size: var(--theme-small, var(--default-theme-small));
  /*  layered box shadow dilenator in case themes don't have borders on their sidebar*/
  box-shadow:
    -1px 0 0 0 var(--theme-border-color, var(--default-theme-border-color)),
    -1px 0 0 0 var(--theme-background-1, var(--default-theme-background-1));
}
@media screen and (max-width: 1000px) {
  .swagger-editor {
    border-right: none;
    box-shadow: none;
  }
}
</style>
