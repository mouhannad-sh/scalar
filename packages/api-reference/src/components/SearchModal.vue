<script setup lang="ts">
import { useKeyboardEvent } from '@scalar/use-keyboard-event'
import { FlowModal, type ModalState } from '@scalar/use-modal'
import Fuse from 'fuse.js'
import { computed, ref, toRef, watch } from 'vue'

import {
  getModelSectionId,
  getOperationSectionId,
  getTagSectionId,
} from '../helpers'
import { extractRequestBody } from '../helpers/specHelpers'
import { type ParamMap, useOperation } from '../hooks'
import type { Spec, TransformedOperation } from '../types'

const props = defineProps<{ parsedSpec: Spec; modalState: ModalState }>()
const reactiveSpec = toRef(props, 'parsedSpec')

type FuseData = {
  title: string
  href: string
  type: 'req' | 'model'
  operationId?: string
  description: string
  body?: string | string[] | ParamMap
  httpVerb?: string
  path?: string
  tag?: string
  operation?: TransformedOperation
}

let fuseDataArray: FuseData[] = []
const searchResults = ref<Fuse.FuseResult<FuseData>[]>([])
const selectedSearchResult = ref<number>(0)
const searchText = ref<string>('')
const searchModalRef = ref<HTMLElement | null>(null)

const fuse = new Fuse(fuseDataArray, {
  keys: ['title', 'description', 'body'],
})

const fuseSearch = (): void => {
  selectedSearchResult.value = 0
  searchResults.value = fuse.search(searchText.value)
}

const selectedEntry = computed<Fuse.FuseResult<FuseData>>(
  () => searchResultsWithPlaceholderResults.value[selectedSearchResult.value],
)

watch(
  () => props.modalState.open,
  (open) => {
    if (!open) return
    searchText.value = ''
    selectedSearchResult.value = 0
    searchResults.value = []
  },
)

watch(
  reactiveSpec.value,
  () => {
    fuseDataArray = []

    if (!props.parsedSpec?.tags?.length) {
      fuse.setCollection([])
      return
    }

    // TODO: We need to go through the operations, not the tags. Spec files can have zero tags.
    props.parsedSpec.tags.forEach((tag) => {
      const tagData: FuseData = {
        title: tag.name,
        href: `#${getTagSectionId(tag)}`,
        description: tag.description,
        type: 'req',
        tag: tag.name,
        body: '',
      }

      fuseDataArray.push(tagData)

      if (tag.operations) {
        tag.operations.forEach((operation) => {
          const { parameterMap } = useOperation({ operation })
          const bodyData = extractRequestBody(operation) || parameterMap.value
          let body = null
          if (typeof bodyData !== 'boolean') {
            body = bodyData
          }

          const operationData: FuseData = {
            type: 'req',
            title: operation.name ?? operation.path,
            href: `#${getOperationSectionId(operation, tag)}`,
            operationId: operation.operationId,
            description: operation.description ?? '',
            httpVerb: operation.httpVerb,
            path: operation.path,
            tag: tag.name,
            operation,
          }

          if (body) {
            operationData.body = body
          }

          fuseDataArray.push(operationData)
        })
      }
    })

    // Adding models as well
    const schemas = props.parsedSpec.components?.schemas
    const modelData: FuseData[] = []

    if (schemas) {
      Object.keys(schemas).forEach((k) => {
        modelData.push({
          type: 'model',
          title: 'Model',
          href: `#${getModelSectionId(k)}`,
          description: k,
          tag: k,
          body: '',
        })
      })

      fuseDataArray = fuseDataArray.concat(modelData)
    }

    fuse.setCollection(fuseDataArray)
  },
  { immediate: true },
)

useKeyboardEvent({
  element: searchModalRef,
  keyList: ['enter'],
  active: () => props.modalState.open,
  handler: () => {
    if (!window) return
    window.location.hash = selectedEntry.value.item.href
    props.modalState.hide()
  },
})

useKeyboardEvent({
  element: searchModalRef,
  keyList: ['ArrowDown'],
  active: () => props.modalState.open,
  handler: () => {
    if (
      selectedSearchResult.value <
      searchResultsWithPlaceholderResults.value.length - 1
    ) {
      selectedSearchResult.value++
    } else {
      selectedSearchResult.value = 0
    }
  },
})

useKeyboardEvent({
  element: searchModalRef,
  keyList: ['ArrowUp'],
  active: () => props.modalState.open,
  handler: () => {
    if (selectedSearchResult.value > 0) {
      selectedSearchResult.value--
    } else {
      selectedSearchResult.value =
        searchResultsWithPlaceholderResults.value.length - 1
    }
  },
})

const searchResultsWithPlaceholderResults = computed(
  (): Fuse.FuseResult<FuseData>[] => {
    if (searchText.value.length === 0) {
      return fuseDataArray.map((item) => {
        return {
          item: item,
        } as Fuse.FuseResult<FuseData>
      })
    }

    return searchResults.value
  },
)
</script>
<template>
  <FlowModal
    :state="modalState"
    variant="search">
    <div
      ref="searchModalRef"
      class="ref-search-container">
      <input
        v-model="searchText"
        class="ref-search-input"
        placeholder="Search …"
        type="text"
        @input="fuseSearch" />
    </div>
    <div
      v-if="searchResultsWithPlaceholderResults.length"
      class="ref-search-list custom-scroll">
      <a
        v-for="(entry, index) in searchResultsWithPlaceholderResults"
        :key="entry.refIndex"
        class="item-entry"
        :class="{
          'item-entry--active': index === selectedSearchResult,
          'item-entry--tag': !entry.item.httpVerb,
        }"
        :href="entry.item.href"
        @click="modalState.hide"
        @focus="selectedSearchResult = index"
        @mouseover="selectedSearchResult = index">
        <div
          class="item-entry-http-verb"
          :class="`item-entry-http-verb--${entry.item.httpVerb}`">
          {{ entry.item.httpVerb }}
        </div>
        <div
          v-if="entry.item.title"
          class="item-entry-title">
          {{ entry.item.title }}
        </div>

        <div
          v-if="
            (entry.item.httpVerb || entry.item.path) &&
            entry.item.path !== entry.item.title
          "
          class="item-entry-path">
          {{ entry.item.path }}
        </div>
        <div
          v-else-if="entry.item.description"
          class="item-entry-description">
          {{ entry.item.description }}
        </div>
      </a>
    </div>
    <div class="ref-search-meta">
      <span>↑↓ Navigate</span>
      <span>⏎ Select</span>
    </div>
  </FlowModal>
</template>
<style scoped>
a {
  text-decoration: none;
}
/** Input */
.ref-search-input {
  width: 100%;
  background: transparent;
  padding: 12px;
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
  outline: none;
  border: 1px solid var(--theme-border-color, var(--default-theme-border-color));
  border-radius: var(--theme-radius, var(--default-theme-radius));
  color: var(--theme-color-1, var(--default-theme-color-1));
  font-weight: var(--theme-semibold, var(--default-theme-semibold));
  font-size: var(--theme-font-size-3, var(--default-theme-font-size-3));
  font-family: var(--theme-font, var(--default-theme-font));
  appearance: none;
}
.ref-search-input:focus {
  border-color: var(--theme-color-1, var(--default-theme-color-1));
}
/** Results */
.item-entry {
  appearance: none;
  background: transparent;
  border: none;
  outline: none;
  padding: 9px 12px;
  width: 100%;
  color: var(--theme-color-3, var(--default-theme-color-3));
  text-align: left;
  border-radius: var(--theme-radius, var(--default-theme-radius));
  display: flex;
  align-items: center;
  font-family: var(--theme-font);
  min-height: 31px;
  display: flex;
  gap: 6px;
  overflow: hidden;
}
.item-entry-http-verb:empty {
  display: none;
}
.ref-search-list {
  padding: 0 0 12px 12px;
}
.ref-search-container {
  padding: 12px;
}
.item-entry--active {
  background: var(--theme-background-2, var(--default-theme-background-2));
  cursor: pointer;
}

/** If it’s a tag, let’s put a dash between the tag name and the description and set the margin to the gap size. */
.item-entry--tag .item-entry-description::before {
  content: '–';
  margin-right: 6px;
}
.item-entry-description,
.item-entry-title {
  font-weight: var(--theme-semibold, var(--default-theme-semibold));
  color: var(--theme-color-1, var(--default-theme-color-1));
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
  white-space: nowrap;
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.item-entry-title {
  min-width: fit-content;
}
.item-entry-http-verb,
.item-entry-subtitle {
  display: flex;
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
  font-family: var(--theme-font-code, var(--default-theme-font-code));
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.item-entry-http-verb {
  text-transform: uppercase;
  min-width: 45px;
  position: relative;
  /* optically center since all characters  above baseline*/
  top: 0.5px;
}
.item-entry-http-verb--post {
  color: var(--theme-color-green, var(--default-theme-color-green));
}
.item-entry-http-verb--patch {
  color: var(--theme-color-yellow, var(--default-theme-color-yellow));
}
.item-entry-http-verb--get {
  color: var(--theme-color-blue, var(--default-theme-color-blue));
}
.item-entry-http-verb--delete {
  color: var(--theme-color-red, var(--default-theme-color-red));
}
.item-entry-http-verb--delete {
  font-size: 0;
}
.item-entry-http-verb--delete:after {
  content: 'DEL';
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
}
.item-entry-http-verb--put {
  color: var(--theme-color-orange, var(--default-theme-color-orange));
}
.item-entry-path {
  color: var(--theme-color-3, var(--default-theme-color-3));
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
  min-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.ref-search-meta {
  background: var(--theme-background-3, var(--default-theme-background-3));
  padding: 6px 12px;
  font-size: var(--theme-font-size-4, var(--default-theme-font-size-4));
  color: var(--theme-color-3, var(--default-theme-color-3));
  font-weight: var(--theme-semibold, var(--default-theme-semibold));
  display: flex;
  gap: 12px;
}
</style>
