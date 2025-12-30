# Vue.js Framework

> **Scope**: Vue.js 3.x applications  
> **Applies to**: .vue files and Vue TypeScript files
> **Extends**: typescript/architecture.md, typescript/code-style.md

## CRITICAL REQUIREMENTS

> **ALWAYS**: Use Composition API with `<script setup>`
> **ALWAYS**: Use `ref()` or `reactive()` for state
> **ALWAYS**: Define props with TypeScript interfaces
> **ALWAYS**: Use computed() for derived state
> **ALWAYS**: Clean up side effects in onBeforeUnmount
> 
> **NEVER**: Use Options API in new code
> **NEVER**: Mutate props directly
> **NEVER**: Access refs without .value in `<script>`
> **NEVER**: Forget to define emits
> **NEVER**: Use `any` type for props/emits

## Core Patterns

### Component with Props & Emits

```vue
<script setup lang="ts">
interface Props {
  modelValue: string
  placeholder?: string
}

interface Emits {
  (e: 'update:modelValue', value: string): void
  (e: 'submit'): void
}

const props = defineProps<Props>()
const emit = defineEmits<Emits>()

function handleInput(event: Event) {
  emit('update:modelValue', (event.target as HTMLInputElement).value)
}
</script>

<template>
  <input :value="modelValue" :placeholder="placeholder" @input="handleInput" />
</template>
```

### Reactive State

```vue
<script setup lang="ts">
import { ref, reactive, computed } from 'vue'

// ref for primitives
const count = ref(0)
const message = ref('Hello')

// reactive for objects
const state = reactive({
  user: { name: 'John', age: 30 },
  items: [1, 2, 3]
})

// computed for derived
const doubled = computed(() => count.value * 2)

function increment() {
  count.value++  // .value required in <script>
}
</script>

<template>
  <button @click="increment">{{ count }}</button>  <!-- No .value in template -->
</template>
```

### Lifecycle & Side Effects

```vue
<script setup lang="ts">
import { onMounted, onBeforeUnmount } from 'vue'

let interval: number

onMounted(() => {
  interval = setInterval(() => console.log('tick'), 1000)
})

onBeforeUnmount(() => {
  clearInterval(interval)
})
</script>
```

### Composables (Reusable Logic)

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initial: number = 0) {
  const count = ref(initial)
  const doubled = computed(() => count.value * 2)
  
  function increment() { count.value++ }
  function decrement() { count.value-- }
  
  return { count, doubled, increment, decrement }
}

// Component.vue
import { useCounter } from './composables/useCounter'
const { count, doubled, increment } = useCounter(10)
```

## Common AI Mistakes

| Mistake | ❌ Wrong | ✅ Correct |
|---------|---------|-----------|
| **Options API** | `data() { return ... }` | `const state = ref()` |
| **No .value** | `count++` in script | `count.value++` |
| **Prop Mutation** | `props.value = x` | `emit('update:value', x)` |
| **any Type** | `props: any` | Interface with types |

## AI Self-Check

- [ ] Composition API with <script setup>?
- [ ] ref()/reactive() for state?
- [ ] TypeScript interfaces for props?
- [ ] computed() for derived?
- [ ] Side effects cleanup?
- [ ] .value in <script>?
- [ ] Emits defined?
- [ ] No prop mutation?
- [ ] No any types?

## Key Features

| Feature | Purpose |
|---------|---------|
| <script setup> | Composition API |
| ref() | Reactive primitives |
| reactive() | Reactive objects |
| computed() | Derived state |
| Composables | Reusable logic |

## Best Practices

**MUST**: Composition API, ref()/reactive(), TypeScript, computed(), cleanup
**SHOULD**: Composables, v-model, provide/inject, Pinia
**AVOID**: Options API, prop mutation, any types, no .value
