# Testing VueApollo Components in the Testing Library style

Since discovering the [Testing Library](https://testing-library.com/), I changed the way I test more and more using its principles:
 - test your application like a user would use it
 - focus on use cases and avoid implementation details
 - only mock if necessary

And I see the benefits pretty clear:
 - Refactoring of components won't (always) break the tests
 - The tests really cover the features of the application and give me more confidence that everything works

## Mock vue-apollo responses
 But there are still some things that can get tricky. For example testing a feature involving data from vue-apollo. Its documentation recommends to [set data directly on the wrapper while testing](https://vue-apollo.netlify.com/guide/testing.html#simple-tests). But this is clearly not how user will interact with the application and it requires implementation details of the internal component data. Besides that the transformation from the graphql response to the actual component data is untested. And even things like loading or error handling won't work without mocking every single call.

```js
const wrapper = mount(App, {
  mocks: {
    $apollo: {
      queries: {
        helloWorld: {
          loading: true,
        },
      },
    },
  },
})
wrapper.setData({
  hello: 'world'
})
```
I expected an interface which I got used to from axios in combination with jest.
```js
axios.get.mockResolvedValue({hello:'world'})
```

Both frameworks get data from somewhere but the tests look very different. Let's take a look how this can be improved with vue-apollo.

The Apollo Smart Queries as well as the components will use an underlying function to send the actual query. This function is also exposed via `this.$apollo.query`. We want to mock those queries and return the right values for our test.

*Disclaimer: To get that working we will create our own wrapper component using the ApolloClient, as the existing apollo tools won't work during test execution. This might be fixed at some point.*

### Successfull Request
Let's start with a new test and how we expect it to look like:
```js
import { render } from "@testing-library/vue";

test("should load and show continents", async () => {
  const { getByText } = render(HelloWorld, {
    mocks: {
      $apollo: {
        query: jest.fn().mockResolvedValue({
          data: { continents: [{ name: "Africa" }] }
        })
      }
    }
  });
  getByText("loading...");
  await flushPromises();
  getByText("Africa");
});
```
To make this test pass, we add the query to our component and show a loading messing until it gets resolved:
```vue
<template>
  <div v-if="loading">
    loading...
  </div>
  <div v-else-if="response">
    <div v-for="continent in response.continents" :key="continent.name">
      {{ continent.name }}
    </div>
  </div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "HelloWorld",
  data() {
    return {
      response: null,
      loading: true,
      getContinents: gql`
        query {
          continents {
            name
          }
        }`
    }
  },
  created() {
    this.$apollo
    .query({
      query: this.getContinents
    })
    .then(({ data }) => {
      this.response = data;
      this.loading = false;
    })
  }
}
```

### Failed Request
This way we can also simulate a failed request in our test environment:
```js
test("should show an error when loading continents fails", async () => {
  const { getByText } = render(HelloWorld, {
    mocks: {
      $apollo: {
        query: jest.fn().mockRejectedValue()
      }
    }
  });
  getByText("loading...");
  await flushPromises();
  getByText("Sorry, there was an error.");
});
```
The implementation could look like the following.
```vue
<template>
  <div v-if="loading">
    loading...
  </div>
  <div v-else-if="response">
    <div v-for="continent in response.continents" :key="continent.name">
      {{ continent.name }}
    </div>
  </div>
  <div v-else-if="error">Sorry, there was an error.</div>
</template>

<script>
import gql from "graphql-tag";

export default {
  name: "HelloWorld",
  data() {
    return {
      response: null,
      loading: true,
      error: false,
      getContinents: gql`
        query {
          continents {
            name
          }
        }`
    }
  },
  created() {
    this.$apollo
    .query({
      query: this.query
    })
    .then(({ data }) => {
      this.data = data;
    })
    .catch(() => {
      this.error = true;
    })
    .finally(() => {
      this.loading = false;
    })
  }
}
```

## Create a testable data fetcher for vue-apollo
So far so good, we are able to load data, show it and handle errors. All of this in a test driven approach. The only drawback so far ist that we gave up the nice interface components which come with apollo. To tackle that we will introduce our own component, specifically tied to our needs. While keeping our tests green all the time, as this refactoring only affects the implementation details.

We will introduce a component which receives a query and provides the response, a loading and error indicator. Our current component will be reduced to the business logic:
```vue
<template>
  <ContinentDataFetcher :query="getContinents">
    <template v-slot="{ response, loading, error }">
      <div v-if="loading">
        loading...
      </div>
      <div v-else-if="response">
        <div v-for="continent in response.continents" :key="continent.name">
          {{ continent.name }}
        </div>
      </div>
      <div v-else-if="error">Sorry, there was an error.</div>
    </template>
  </ContinentDataFetcher>
</template>

<script>
import gql from "graphql-tag";
import ContinentDataFetcher from "./ContinentDataFetcher";

export default {
  name: "HelloWorld",
  components: { ContinentDataFetcher },
  data() {
    return {
      getContinents: gql`
        query {
          continents {
            name
          }
        }
      `
    };
  }
};
</script>
```
The extracted code belongs to a new data fetcher or data provider component. This components just provides some data and will not render and DOM elements, it is a so called [renderless component](https://adamwathan.me/renderless-components-in-vuejs/). Above we can already see how to use it, so lets take a look at its implementation:

```vue
<script>
export default {
  name: "ContinentDataFetcher",
  props: ["query"],
  data() {
    return {
      data: null,
      error: false,
      loading: true
    };
  },
  created() {
    this.$apollo
      .query({
        query: this.query
      })
      .then(({ data }) => {
        this.data = data;
      })
      .catch(() => {
        this.error = true;
      })
      .finally(() => {
        this.loading = false;
      });
  },
  render() {
    return this.$scopedSlots.default({
      loading: this.loading,
      response: this.data,
      error: this.error
    });
  }
};
</script>
```
This gives us a reusable component which can be easily tested and customized. Feel free to add some modifications like *a data transformation property which will transform the returned data* or *a general preview mechanism which will add variables to your queries based on a global flag.*

## An example using vue-test-utils
Searching for the fixed tests? Sorry, no change needed. But in case you want an example using the vue-tets-utils, here you go:
```js
import { mount } from '@vue/test-utils'

test("should load and show continents", async () => {
  const wrapper = mount(HelloWorld, {
    mocks: {
      $apollo: {
        query: jest.fn().mockResolvedValue({
          data: { continents: [{ name: "Africa" }] }
        })
      }
    }
  });
  expect(wrapper.html()).toContain('loading...')
  await flushPromises();
  expect(wrapper.html()).toContain('Africa')
})
```
