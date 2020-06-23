<style lang="postcss" scoped>
nav.pagination button {
  @apply -ml-px;
  @apply relative;
  @apply inline-flex;
  @apply items-center;
  @apply px-4;
  @apply py-2;
  @apply border;
  @apply border-gray-300;
  @apply bg-white;
  @apply text-sm;
  @apply font-medium;
  @apply text-gray-700;

  line-height: 1.25rem;
  transition-property: background-color, border-color, color, fill, stroke,
    opacity, box-shadow, transform;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 150ms;
}

nav.pagination button:hover {
  @apply bg-gray-500;
  @apply text-gray-100;
}

nav.pagination button:focus {
  @apply z-10;
  @apply outline-none;

  box-shadow: 0 0 0 1px #4a5568;
}

nav.pagination button.active {
  @apply bg-gray-700;
  @apply text-gray-100;
}

nav.pagination button.spec {
  @apply px-2;
  @apply text-gray-700;
}

nav.pagination button.spec.prev {
  @apply rounded-l-lg;
}

nav button.spec.next {
  @apply rounded-r-lg;
}

nav.pagination button.spec:hover {
  @apply bg-gray-400;
}

nav.pagination span {
  @apply -ml-px;
  @apply relative;
  @apply inline-flex;
  @apply items-center;
  @apply px-4;
  @apply py-2;
  @apply border;
  @apply border-gray-300;
  @apply bg-white;
  @apply text-sm;
  @apply font-medium;
  @apply text-gray-700;

  line-height: 1.25rem;
}
</style>

<template>
  <nav class="pagination relative z-0 inline-flex shadow-sm">
    <button type="button" class="spec prev" aria-label="Previous" @click="setPrev">
      <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        <path
          fill-rule="evenodd"
          d="M12.707 5.293a1 1 0 010 1.414L9.414 10l3.293 3.293a1 1 0 01-1.414 1.414l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 0z"
          clip-rule="evenodd"
        />
      </svg>
    </button>

    <template v-for="(button, index) of buttons">
      <button
        type="button"
        :key="'button-'+index"
        v-if="button != '...'"
        :class="(button == page ? 'active' : null)"
        @click="$emit('change', button)"
      >{{button}}</button>
      <span v-else :key="'span-'+index">...</span>
    </template>

    <button type="button" class="spec next" aria-label="Next" @click="setNext">
      <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
        <path
          fill-rule="evenodd"
          d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
          clip-rule="evenodd"
        />
      </svg>
    </button>
  </nav>
</template>

<script>
export default {
  name: "pagination",
  model: {
    event: "change",
    prop: "page"
  },
  props: {
    page: Number,
    maxPage: Number
  },
  computed: {
    buttons() {
      if (this.maxPage <= 7)
        return Array.from(Array(this.maxPage), (_, i) => i + 1);
      else if (this.page <= 4) return [1, 2, 3, 4, 5, "...", this.maxPage];
      else if (this.page >= this.maxPage - 3)
        return [
          1,
          "...",
          this.maxPage - 4,
          this.maxPage - 3,
          this.maxPage - 2,
          this.maxPage - 1,
          this.maxPage
        ];
      else
        return [
          1,
          "...",
          this.page - 1,
          this.page,
          this.page + 1,
          "...",
          this.maxPage
        ];
    }
  },
  methods: {
    setPrev() {
      if (this.page > 1) this.$emit('change', this.page - 1);
    },
    setNext() {
      if (this.page < this.maxPage) this.$emit('change', this.page + 1);
    }
  }
};
</script>
