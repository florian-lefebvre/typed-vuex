---
title: Dynamic modules
description: 'Vanilla, strongly-typed store accessor.'
category: Accessor
position: 11
---

You can also use `typed-vuex` with dynamic modules.

## Sample module

```ts{}[modules/dynamic-module.ts]
export const namespaced = true

export const state = () => ({
  emails: [] as string[],
})

export const mutations = mutationTree(state, {
  addEmail(state, newEmail: string) {
    state.emails.push(newEmail)
  },
})
```

## Accessing the module

You might want to use the store

```ts{}[components/my-component.vue]
import Vue from 'vue

import { useAccessor, getAccessorType } from 'typed-vuex'
import dynamicModule from '~/modules/dynamic-module'

const accessorType = getAccessorType(dynamicModule)

export default Vue.extend({
  data: () => ({
    accessor: null as typeof accessorType | null,
  }),
  beforeCreated() {
    // make sure the namespaces match!
    this.$store.registerModule('dynamicModule', dynamicModule, {
      preserveState: false,
    })
    const accessor = useAccessor(this.$store, dynamicModule, 'dynamicModule')

    // this works and is typed
    accessor.addEmail('my@email.com')

    // but you might want to save the accessor for use elsewhere in your component
    this.accessor = accessor
  },
  methods: {
    anotherMethod() {
      // ... such as here
      if (this.accessor) {
        this.accessor.addEmail('my@email.com')
      }
    }
  }
})

```
