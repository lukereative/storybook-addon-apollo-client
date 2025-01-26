# Storybook Addon Apollo Client

Use Apollo Client in your Storybook stories.

## Versions

- If you're using Apollo Client 2.x and Storybook 5.x use version 1.x
- If you're using Apollo Client 2.x or 3.x and Storybook 6.x use version 4.x
- If you're using Apollo Client 2.x or 3.x and Storybook 7.x use version 5.x
- If you're using Apollo Client 2.x or 3.x and Storybook 8.x use version 7.x
- If you're using Apollo Client 3.x and Storybook 8.3+ use version 8.x

## Known issues

due to how MockedProvider works in Apollo, you will have to hard refresh when visiting sub stories (visiting stories that exist in the same file) to get the expected results, please upvote my comment here to see if we can get this fixed: https://github.com/apollographql/apollo-client/issues/9738#issuecomment-1606316338

## Install

**pnpm**

```shell
pnpm add -D storybook-addon-apollo-client
```

**yarn**

```shell
yarn add -D storybook-addon-apollo-client
```

**npm**

```shell
npm install -D storybook-addon-apollo-client
```

Add the addon to your configuration in `.storybook/main.ts`

```js
export default {
  ...config,
  addons: [
    ...yourAddons
    "storybook-addon-apollo-client",
  ],
};
```

## 7.0 Features

- removed `globalMocks` key to favor composition

### Migrate from 5.x+ to 7.x

#### Example of < 7.x

**preview.ts**

```js
import { MockedProvider } from "@apollo/client/testing"; // Use for Apollo Version 3+
// import { MockedProvider } from "@apollo/react-testing"; // Use for Apollo Version < 3

export const preview = {
  parameters: {
    apolloClient: {
      MockedProvider,
      globalMocks: [
        // whatever mocks you want here
      ],
    },
  },
};
```

#### Example of 7.x

**preview.ts**

```js
// Whatever you want here, but not Apollo Client related
```

**component.stories.ts**

```ts
import type { Meta } from "@storybook/react";
import { globalMocks } from "./globalMocks";
import { otherMocks } from "./otherMocks";
import { YourComponent, YOUR_QUERY } from "./component";

export const meta: Meta<typeof DisplayLocation> = {
  component: YourComponent,
  parameters: {
    apolloClient: {
      mocks: [
        ...globalMocks,
        ...otherMocks,
        {
          request: {
            query: YOUR_QUERY,
          },
          result: {
            data: {
              // your data here
            },
          },
        },
      ],
    },
  },
};
```

## Upgrading from a previous version below 6.x

In previous versions, we had a decorator called `withApolloClient` this is no longer nesscessary. If you're upgrading from this API here are the following changes that you'll need to make:

1. remove all code referencing the deprecated withApolloClient decorator.
2. follow install instructions

## Writing your stories with queries

```jsx
import DashboardPage, { DashboardPageQuery } from ".";

export default {
  title: "My Story",
};

export const Example = () => <DashboardPage />;

Example.parameters = {
  apolloClient: {
    mocks: [
      {
        request: {
          query: DashboardPageQuery,
        },
        result: {
          data: {
            viewer: null,
          },
        },
      },
    ],
  },
};
```

Read more about the options available for MockedProvider at https://www.apollographql.com/docs/react/development-testing/testing

### Usage

In Storybook, click "Show Addons" and navigate to the "Apollo Client" tab.

![Addon UI Preview](preview.png)

## Example App

To see real world usage of how to use this addon, check out the example app:

https://github.com/lifeiscontent/realworld

## Loading State

You can use the `delay` parameter to simulate loading state.

```js
import DashboardPage, { DashboardPageQuery } from ".";

export default {
  title: "My Story",
};

export const Example = () => <DashboardPage />;

Example.parameters = {
  apolloClient: {
    mocks: [
      {
        // Use `delay` parameter to increase loading time
        delay: 1000,
        request: {
          query: DashboardPageQuery,
        },
        result: {
          data: {},
        },
      },
    ],
  },
};
```

## Error State

You can use the `error` parameter to create error state.

```js
import DashboardPage, { DashboardPageQuery } from ".";

export default {
  title: "My Story",
};

export const Example = () => <DashboardPage />;

Example.parameters = {
  apolloClient: {
    mocks: [
      {
        request: {
          query: DashboardPageQuery,
        },
        error: new ApolloError("This is a mock network error"),
      },
    ],
  },
};
```


### Vue Support

In order to use this plugin with [@storybook/vue3](https://www.npmjs.com/package/@storybook/vue3):

Add the addon to your configuration in `.storybook/main.js`

```js
module.exports = {
  ...config,
  addons: [
    ...your addons
    "storybook-addon-apollo-client/preset-vue",
  ],
};
```

create a component in `.storebook/MockedProvider.vue`
```vue
<script setup lang="ts">
import { ApolloClient, DefaultOptions } from '@apollo/client/core';
import { InMemoryCache as Cache } from '@apollo/client/cache';
import { MockLink, MockedResponse } from '@apollo/client/testing/core';
import type { ApolloLink } from '@apollo/client/link/core';
import type { Resolvers } from '@apollo/client/core';
import type { ApolloCache } from '@apollo/client/cache';
import {DefaultApolloClient} from "@vue/apollo-composable";
import {provide} from "vue";

export interface MockedProviderOptions<TSerializedCache = {}> {
    mocks?: ReadonlyArray<MockedResponse>;
    addTypename?: boolean;
    defaultOptions?: DefaultOptions;
    cache?: ApolloCache<TSerializedCache>;
    resolvers?: Resolvers;
    childProps?: object;
    children?: any;
    link?: ApolloLink;
}

const {
    mocks,
    addTypename,
    defaultOptions,
    cache,
    resolvers,
    link
} = defineProps<MockedProviderOptions>();

const client = new ApolloClient({
    cache: cache || new Cache({ addTypename }),
    defaultOptions,
    link: link || new MockLink(
        mocks || [],
        addTypename,
    ),
    resolvers,
});

provide(DefaultApolloClient as Symbol, client);
</script>

<template>
    <slot></slot>
</template>
```

> **_NOTE:_**  If someone with time could publish the component as a separate package installation and usage could be simplified

add the following to your `.storybook/preview.js`

```js
import MockedProvider from './MockedProvider'; // Use for Apollo Version 3+

export const parameters = {
  apolloClient: {
    MockedProvider,
    // any props you want to pass to MockedProvider on every story
  },
};
```

any other instruction will work as expected.
