<script setup lang="ts">
import { ThemeStyles } from '@scalar/themes'

import PageLink from '../components/PageLink.vue'
</script>
<template>
  <ThemeStyles id="default" />
  <div class="main dark-mode">
    <div class="logo">
      <svg
        data-v-873746d2=""
        height="50"
        viewBox="0 0 593 593"
        width="50"
        xmlns="http://www.w3.org/2000/svg">
        <path
          d="M347 0c6 0 12 5 12 12v134l94-95c5-5 13-5 17 0l72 72c4 4 5 12 0 16v1l-95 94h134c7 0 12 5 12 12v101c0 7-5 12-12 12H447l95 94c4 5 5 13 0 17l-72 72c-4 4-12 5-16 0h-1l-94-95v134c0 7-5 12-12 12H246c-7 0-12-5-12-12v-70c0-22 9-43 24-59l130-130c14-14 14-37 0-51L259 142a84 84 0 0 1-25-59V12c0-7 5-12 12-12h101ZM138 52h1l219 219c14 14 14 37 0 51L139 542c-4 5-12 5-17 0l-71-70c-4-5-5-12 0-17l95-96H12c-7 0-12-5-12-12V246c0-7 5-12 12-12h134l-95-94c-4-5-4-12 0-17l71-71c4-5 12-5 16 0Z"
          fill="currentColor"
          fill-rule="evenodd"></path>
      </svg>
    </div>
    <h1>Examples</h1>
    <div class="page-links">
      <PageLink to="standalone-api-reference">
        <template #title>Full API Reference</template>
        <template #description>
          Interactive API documentation with pre-configured search and all the
          bells and whistles.
        </template>
      </PageLink>
      <PageLink to="classic-api-reference">
        <template #title>Classic API Reference</template>
        <template #description>
          Classic API documentation with search. Looks like Swagger UI if it was
          born today.
        </template>
      </PageLink>
    </div>
    <h1>Packages</h1>
    <div class="page-links">
      <PageLink to="api-reference">
        <template #title>API Reference</template>
        <template #description>
          The customizable interactive API documentation, pretty bare bones if
          you ask me.
        </template>
      </PageLink>
      <PageLink to="api-client">
        <template #title>API Client</template>
        <template #description>
          Just the API client, and nothing else. Play with any API in your
          browser.
        </template>
      </PageLink>
      <PageLink to="swagger-editor">
        <template #title>Swagger Editor</template>
        <template #description>
          Our OpenAPI/Swagger editor, and just that. Ready to be embedded in any
          website.
        </template>
      </PageLink>
    </div>
  </div>
</template>

<style scoped>
.main {
  font-family: 'Inter', sans-serif;
  background-color: var(--default-theme-background-1);
  color: var(--default-theme-color-1);
  min-height: 100vh;
  padding: 2rem;
  box-sizing: border-box;
}

h1 {
  font-weight: 300;
  margin-bottom: 2rem;
  margin-top: 4rem;
  font-size: 1.8rem;
}

.page-links {
  display: grid;
  grid-template-columns: repeat(auto-fit, 350px);
  grid-gap: 1rem;
  /** Make all PageLink elements same height */
}
</style>
