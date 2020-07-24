# A short glimpse on AWS amplify and Google Firebase

AWS Amplify and Google Firebase are two responses to the need for fast and scaling application development. 
The provided services can help to evaluate ideas quickly while serving near endless amount of traffic.

## What can Amplify do for us?
What impressed me the most is the GraphQL API. 
You just need to [define the GraphQL schema and Amplify will create the whole backend](https://docs.amplify.aws/cli/graphql-transformer/overview). 
You will also get a set of queries, mutations and subscriptions to work with your data from your frontend.
This might not work for every use case right away. But if your app should enable users to create, display and update data. This might be all you need.

Another thing kind of every app needs is User Management. 
Amplify provides not only services to tackle this, but also a set of default components which you can use in your application.
Registration, Login, Logout are solved problems. 
Better focus on generating real bussiness value. 

## What Amplify will not do for us?
You will still have to impement your business logic and say your application what to do with this data.
Creating nice user interfaces will also stay under your control. From handling form inputs to styling.

## How is Firebase different?
Firebase will also provide a lot of services for common tasks like user management.
But it is different when it comes to data. You neither will be able to create a graphql schema, nor is it even needed.
Using firebase feels like [talking directly to a NoSQL database](https://firebase.google.com/docs/firestore/query-data/get-data), from your frontend. 
But this also comes with the burden to structure your data wisely. 
There is no schema that forces you to be consistent. 
And there are no premade queries or mutations to start with.

## When to use what?
You better take a closer look at both of them before making a decision. Or just get started and try them. :)

One thing you might get out of this glimpse is to think about your data structure. Would you benefit from typed data model?
