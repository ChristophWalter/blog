# Using webpack code and bundle splitting with vue.js

## Bundle Splitting
> helps to create more smaller files which improves caching as updating one package will not affect others. This can be used to split out dependencies or big independent parts of your application. **CACHE UNCHANGED CODE**

You can manually configure to split your application into parts using *entry points* in your webpack configuration. Shared dependencies between your entry points can furthermore be configured within *cacheGroups*. This is where you can also split out every npm package by giving them an individual name using a function.

*chunks: all* will allow you to share chunks between async and non-async chunks. Read more about async chunks in the code splitting block.

```js
module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            // return a chunk name per package if it should be separated
            return `vendor-${module.context}``
          },
        },
      },
    },
  },
};
```

## Code Splitting
> loads code dynamically once it is needed. This is how you can improve the time to load a single page as you may have lots of code or dependencies which are not used there.
**ONLY LOAD CODE YOU NEED**

This can be done within your vue application. For example you can load a component with all its dependencies as separate chunk. This makes sense when using big libraries like swagger-ui, a library to render api specification.

Just use webpacks import() instead of import. Combined with vue Single File Components this will look like the following:
```vue
<script>
export default {
  components: {
    Tryout: () => import(/* webpackChunkName: "Example" */ '@/components/HelloWorld'),
  },
}
</script>
```
You can also import the third party library itself async instead of the surrounding component. But keep in mind that import() returns a promise and you need to handle that when working with the library and within your tests. Using it like above with SFC will do it for you.

You may need to allow comments in your babel configuration to pass a chunk name.

To support async imported components within your jests tests you will need the `babel-plugin-dynamic-import-node`.

Further readings:
https://hackernoon.com/the-100-correct-way-to-split-your-chunks-with-webpack-f8a9df5b7758
