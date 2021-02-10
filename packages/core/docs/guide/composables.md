## Composition API

> If you already have an experience with Vue Composition API, it's safe to skip this section and start with [Vue Storefront Composables](#what-are-vue-storefront-composables)

Composition API is a new way to abstract and reuse the logic added in Vue 3.0. It allows you to create and observe a reactive state both inside the Vue component and outside it as a standalone function.

Let's try to build functionality for submitting a form with two fields: user name and password.

```html
<template>
  <form>
    <input type="text" placeholder="Username" />
    <input type="password" placeholder="Password" />
    <button type="submit">Submit</button>
  </form>
</template>
```

In the Vue component, Composition API methods should be called inside the new component option called `setup`:

```html
<script>
  export default {
    setup() {
      // here we will start using Composition API
    },
    data() {
      return {
        /* ... */
      };
    },
    methods: {
      /* ... */
    }
  };
</script>
```

First of all, we need two reactive values that will be bound to form fields. They can be created with Vue `ref` method:

```js
import { ref } from 'vue';

export default {
  setup() {
    const username = ref('');
    const password = ref('');

    return { username, password };
  }
};
```

The argument passed to `ref` is an _initial value_ of our reactive state properties. Under the hood, `ref` creates an object with a single property `value`:

```js
setup() {
  const username = ref('');
  const password = ref('');

  console.log(username.value) // => ''

  return { username, password };
}
```

After we returned reactive properties from the `setup`, they become available in Vue component options (such as `data` or `methods`) and in component template. `refs` returned from setup are automatically unwrapped - this means you don't access the `.value` anymore:

```html
<template>
  <form @submit="submitForm">
    <input v-model="username" type="text" placeholder="Username" />
    <input v-model="password" type="password" placeholder="Password" />
    <button type="submit">Submit</button>
  </form>
</template>

<script>
  import { ref } from 'vue';

  export default {
    setup() {
      const username = ref('');
      const password = ref('');

      return { username, password };
    }
    methods: {
      submitForm() {
        console.log(this.username)
      }
    }
  };
</script>
```

Methods can be also created inside the `setup` option:

```js{6-11}
export default {
  setup() {
    const username = ref('');
    const password = ref('');

    const submitForm = () => {
      // remember that inside `setup` you need to access ref's value
      console.log(username.value);
    };

    return { username, password, submitForm };
  }
};
```

Let's disable a button when username or password is empty. In order to do this, we can create a computed property - and this can be also done with the Composition API inside the `setup`:

```html{5,15-17}
<template>
  <form @submit="submitForm">
    <input v-model="username" type="text" placeholder="Username" />
    <input v-model="password" type="password" placeholder="Password" />
    <button :disabled="!isValid" type="submit">Submit</button>
  </form>
</template>

<script>
  import { ref, computed } from 'vue';

  export default {
    setup() {
      /*...*/
      const isValid = computed(
        () => username.value.length > 0 && password.value.length > 0
      );

      return { username, password, submitForm, isValid };
    }
  };
</script>
```

Now, we can make a final step and abstract all the form functionality _outside_ the component - in a standalone function:

```js
// useForm.js

import { ref, computed } from 'vue';

export function useForm() {
  const username = ref('');
  const password = ref('');

  const submitForm = () => {
    console.log(username.value);
  };

  const isValid = computed(
    () => username.value.length > 0 && password.value.length > 0
  );

  return { username, password, submitForm, isValid };
}
```

This function is what we call a **composable**: a reusable piece of logic with the reactive state. Composables can be imported and used in Vue components:

```js
// SomeComponent.vue

import { useForm } from './useForm.js';

export default {
  setup() {
    const { username, password, submitForm, isValid } = useForm();

    return { username, password, submitForm, isValid };
  }
};
```

Vue Storefront uses composables as its main API. We will take a look over them in the next section.

## What are Vue Storefront composables

Composable is a function that uses [Composition API](#composition-api) under the hood. Composables are the main public API of Vue Storefront and, in most cases, the only API except configuration you will work with.

You can treat composables as independent micro-applications. They manage their own state, handle server-side rendering, and rarely interact with each other (except [useUser](/composables/use-user.html) and [useCart](/composables/use-cart.html)). No matter what integration you are using, your application will always have the same set of composables with the same interfaces.

To use a composable, you need to import it from an integration you are using, and call it on the component `setup` option:

```js
import { useProduct } from '{INTEGRATION}';
import { onSSR } from '@vue-storefront/core`

export default {
  setup() {
    const { products, search} = useProduct('<UNIQUE_ID>');

    onSSR(async () => {
      await search(searchParams)
    })

    return {
      products
    };
  }
};
```
> `onSSR` is used to perform an asynchronous request on the server side and convey the received data to the client. You will learn more about it very soon.

For some composables (like `useProduct` ) you will need to pass a unique ID as a parameter (it can be a product ID, category ID etc.). Others (like `useCart`) do not require an ID passed. You can always check a composable signature in the [API Reference](TODO)