---
title: Getting started (Vue)
description: 'Vanilla, strongly-typed store accessor.'
category: Getting started
position: 3
---

If you would like to benefit from a typed accessor to your store, but you're not using Nuxt, you can still use `typed-vuex`.

<d-alert>

Many of this project's default settings are based on Nuxt, so please file an issue if you experience any problems.

</d-alert>

## Setup

1. Install package:

   <d-code-group>
    <d-code-block label="Yarn" active>

    ```bash
    yarn add typed-vuex
    ```

    </d-code-block>
    <d-code-block label="NPM">

    ```bash
    npm install typed-vuex --save
    ```

    </d-code-block>
   </d-code-group>

2. Instantiate your accessor

   ```ts{}[src/store/index.ts]
   import Vue from 'vue'
   import Vuex from 'vuex'

   import {
     useAccessor,
     getterTree,
     mutationTree,
     actionTree,
   } from 'typed-vuex'

   Vue.use(Vuex)

   const state = () => ({
     email: '',
   })

   const getters = getterTree(state, {
     email: state => state.email,
     fullEmail: state => state.email,
   })

   const mutations = mutationTree(state, {
     setEmail(state, newValue: string) {
       state.email = newValue
     },

     initialiseStore() {
       console.log('initialised')
     },
   })

   const actions = actionTree(
     { state, getters, mutations },
     {
       async resetEmail({ commit }) {
         commit('setEmail', 'a@a.com')
       },
     }
   )

   const storePattern = {
     state,
     mutations,
     actions,
   }

   const store = new Vuex.Store(storePattern)

   export const accessor = useAccessor(store, storePattern)

   // Optionally, inject accessor globally
   Vue.prototype.$accessor = accessor

   export default store
   ```

3. Define types.

   If you've injected the accessor globally, you'll want to define its type:

   ```ts{}[index.d.ts]
   import Vue from 'vue'
   import { accessor } from './src/store'

   declare module 'vue/types/vue' {
     interface Vue {
       $accessor: typeof accessor
     }
   }
   ```

## Usage within a component

```ts
import { Component, Vue } from 'vue-property-decorator'
import { accessor } from '../store'

@Component
export default class SampleComponent extends Vue {
  get email() {
    // This (behind the scenes) returns getters['email']
    return accessor.email

    // Or, with a globally injected accessor
    return this.$accessor.email
  }

  resetEmail() {
    // Executes dispatch('submodule/resetEmail', 'new@email.com')
    accessor.submodule.resetEmail('new@email.com')

    // Or, with a globally injected accessor
    this.$accessor.submodule.resetEmail('new@email.com')
  }
}
```

## Usage within the store

You can use the accessor within the store or a store module.

```ts
import { actionTree } from 'typed-vuex'
import { accessor } from '.'

const actions = actionTree(
  { state, getters, mutations },
  {
    async resetEmail({ commit }) {
      accessor.submodule.initialise()
      commit('setEmail', 'a@a.com')
    },
  }
)
```
