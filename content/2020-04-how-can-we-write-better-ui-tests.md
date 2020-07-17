# How can we write better UI Tests?

## What we did
Vue-test-utils recommends to focus on isolated unit testing with a vue component as unit.
> In unit tests, we typically want to focus on the component being tested as an isolated unit and avoid indirectly asserting the behavior of its child components.
>
> <cite>https://vue-test-utils.vuejs.org/guides/common-tips.html</cite>

And thats mostly what we did for some time. In Addition with some e2e testing using Nightwatch and Browserstack.

So let's start with an example on how to test something using this guideline. Imagine a dropdown with a list of cars which gathers and shows data when a user selects one car.

```javascript
// DropdownComponent.spec.js
it('should emit the carID when dropdown is selected',...)
// CarSelectionComponent.spec.js
it('should pass data for the emitted carID to be displayed',...)
// DisplayComponent.spec.js
it('should show the passed car information',...)
```

## THE Problem*s

- There is an additional hurdle to refactor code, as it often breaks the test even though the feature still works.
- Sometimes the unit tests are perfectly green, but the code together still does not work :(
- Tests sometimes are very simple like show this text if it is passed as property, which might lead to not test such framework logic.
- If multiple components interact with some data, every component tests its public interface (e.g. props, events, child components) which is annoying if data is just passed through.
- Testing sometimes gets tricky if you used advanced component practices like scoped slots.
- We tested the interface to some external libraries. But after updating those we can not be sure if everything still works.

## Trying to find a possible solution
All these problems are nothing new, so I gathered some information from the outside world. Happily @maik told me to take a look at Kent C. Dodds, his opinion on ui testing and his React Testing library, so I did. Then I found more and more information like [this talk](https://www.youtube.com/watch?v=EZ05e7EMOLM). Some of the common guidelines from this research are:

1. Do not test implementation details so your tests will not break when you refactor.
2. Focus on features of your application and write tests for those.
3. Write your tests like a user who is using the application.
4. Only mock if necessary like for expensive operations.

Coming back to our car selection example to see how this will affect your tests. As this is one feature, we will write one test case for it. Furthermore we avoid mocking subcomponents like the DropdownComponent and DisplayComponent. We still might mock http calls like getting the car information from our backend server. Our test case could look like the following:

```js
// CarSelection.spec.js
it('should show the car information for the selected car',...)
```

 Taking a look at this testcase we can think about the benefits. We now might refactor the implementation of this feature using new components, patterns or whatever. As long as it stays the same for our users, the test should stay green as well. We also do not have to care about testing implementation details when use fancy (or simple) component patterns, as it does not matter for our test how things are implemented as long as they work. And even our integration of an external libraries will be tested after an update.

 Whether we should now call this unit or integration tests does not matter that much. Neither is it an all or nothing approach. It might still be useful to write test with a smaller scope instead of the whole flow of a feature. Like testing input validation of a form field. In conclusion there is a strong potential when writing tests with these guidelines in the mind. It helps us to avoid some of our current problems (and probably will generate new ones, we will see).

## Trying new approaches
This is how I integrated these guidelines into my development workflow:
 - Red: Write a test for your main feature and how a user would use it.
 - Green: Make the test pass as quick as possible.
 - Refactor: Make your code nice and cross your thumbs that the test stays green.
 - Continue with additional flows like Error handling, cancel, whatever. Or add tests for some details of the feature.

=> Writing this kind of tests even helps me to really focus on the current stage, which is great.
