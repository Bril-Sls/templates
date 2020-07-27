# Refactoring to Webpack

- [Walkthrough Video](https://www.loom.com/share/3b32f103e0314f3d9485b851919ece8e)
- [Discussion about Webpack, Private NPM Modules, and Lambda Layers, and when to use each](https://www.loom.com/share/f543f3c54a564be2afba22fbd5ca8716)

# Which tool does the best job
The following tools:
- webpack
- npm modules
- Lambda Layers

Are tools that are often used to solve 2 different problems:
- Code Reuse (time and maintainability optimization)
- Lambda Function Artifact Size (technical optimization)


## When to use Webpack
In this pattern we are using webpack to solve the problem of npm module size. Optimizing the size of npm modules
is something we suggest you always do. This removes the need to have individual `package.json` files in handler
folders and managing npm modules per function. This also makes referencing util and helper files outside of the
handler folder much easier

## When to use Private NPM Modules
Private NPM modules are great for sharing business logic amount many services. Business logic usually is not
very big. For reference, a 300 line node js file is roughly 10kb. Although they are not big, they represent a lot
of work and a lot of thinking and testing. To keep our services clean and to keep developers from recreating the 
wheel and spending lots of time building the same thing, it is great to instead create private NPM modules
that the organization can share and reuse in services. 

Some important notes about NPM modules
- You can have as many as you want
- The code in an NPM modules is included when packaging your Lambda Function Artifact

## When to use Lambda Layers
Lambda Layers are another way to include already written code for your Lambda Function. The difference here is:
- You can only have 5 per Lambda Function
- Code included in the Lambda Layer is not included in your Lambda Function Artifact

Because you have a limited number of Layers to use, and because they enable your function to be deployed without code
contained in the Layer, Layers are create for very large amounts of code. Some good use cases:

- Swift Runtime Layer (make a Lambda Layer with a Swift runtime so you can write Lambda Functions with Swift)
- Image Processing Layer (sharp is a very large npm module. Making a general Image Processing layer for dependencies like
  these is a great way to keep Lambda Functions light)
- Monitoring Layer (Dynatrace is another example of a heavy npm module)

Because you can only have 5, we encourage you to make large categories of Lambda Layers, rather than small utility 
Lambda Layers. What we dont encourage is the following:

- Dynatrace Layer
- CloudWatchLogger Layer
- ServiceA Layer
- ServiceB Layer

All of these are too small and specific. Instead, we should make a general Monitoring Layer which contains both
dynatrace and any custom code that is related to monitoring. ServiceA and ServiceB are too specific in terms of
their ability to be reused. Instead, those Layers should be broken up 
- 1. If they are business logic, break into small private NPM modules which can 
be used by many services
- 2. If they are very big third party NPM modules or binaries, add them to a Lambda Layer that represents a large
category or usecase.

### Summary
Lambda Layers should represent big categories that everyone can use. NPM modules should represent small utilities or
bussiness logic modules that can be mixed and matched per service to fit its needs. 