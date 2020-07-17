# Testing pending states of promises with Jest

As a user I expect to get instant feedback for an action. I want to know that something is happening an I do not have to do anything for now.

So basically if I submit a form I want to see a loading spinner. And this what we are going to write a test for now.

I am currently playing around with the Vue Testing Library. So here is my first try. We will mock axios, as we do not want to call real APIs in this unit test. So every http post request using axios gets resolved. Perfect.

```javascript
jest.mock('axios')

it('should show a loading spinner while the http request is pending', async () => {
  axios.post.mockResolvedPromise()
  const { getByText } = render(component)
  fireEvent.click(getByText('submit'))
  expect(axios.post).toHaveBeenCalledWith('/submit')
  // wait till Vue updates its DOM
  await Vue.nextTick()
  getByText('loading...')
  getByText('success')
})
```

But wait. We did nothing between asserting for the loading and the success state. So this will fail. And it does. But not like we expected it to. My first thought was it will still be loading and we need to wait for something else till the promise is resolved. But no, after `Vue.nextTick()` it is already in success state.

So let's try something different. We delay waiting for the next tick and use a function of the Vue Testing Library which waits till an element is visible:

```javascript
jest.mock('axios')

it('should show a loading spinner while the http request is pending', async () => {
  axios.post.mockResolvedPromise()
  const { getByText } = render(component)
  fireEvent.click(getByText('submit'))
  expect(axios.post).toHaveBeenCalledWith('/submit')
  await findByText('loading...')
  await Vue.nextTick()
  getByText('success')
})
```

This fails after a timeout, as findByText can not find the loading state as well. The state still jumps directly from initial to success. But why?

There is this concept of [scheduling tasks and microtasks](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) which Browser use when executing JavaScript. Both, promises callbacks (like the mocked promise from axios) [as well as `Vue.nextTick()` will schedule a microtask](https://vue-test-utils.vuejs.org/guides/testing-async-components.html). The microtask queue is processed first-in-first-out. As our mock for the axios call is already a resolved promise, it will add its callback to the microtask queue once the component gets initialised. We can only call `Vue.nextTick()` after mounting the component, so its microtask will always be behind the promise one. Once the DOM gets rerendered, the state will therefore always be success.

## The required change is not resolving the axios promise before the component renders its loading state

And this is how we do it:

```JavaScript
jest.mock('axios')

it('should show a loading spinner while the http request is pending', async () => {
  let resolvePromise
  axios.post.mockResolveValueOnce(
      new Promise(resolve => {
          resolvePromise = resolve
      })
  )
  const { getByText } = render(component)
  fireEvent.click(getByText('submit'))
  expect(axios.post).toHaveBeenCalledWith('/submit')
  await findByText('loading...')
  resolvePromise()
  getByText('success')
})
```
A little bit more boilerplate is needed to resolve the axios promise from outside. But now we control it and finally it works. :)

Are you showing loading states to your users? How did you manage to test them?


By the way: I tried to create a codesandbox for this. But there seems to be problems with testing vue components... if you want to help, take a look here: https://github.com/codesandbox/codesandbox-client/issues/2058
