# Table of Contents

- [What is this?](#what-is-this-)
- [Why?](#why-)
- [Practices](#practices)
  * [Typescript](#typescript)
    + [Pros/cons](#pros-cons)
    + [Should writing more code really be classified as a drawback?](#should-writing-more-code-really-be-classified-as-a-drawback-)
  * [Redux](#redux)
    + [Handling asynchronous actions](#handling-asynchronous-actions)
    + [Structure](#structure)
  * [Project structure](#project-structure)
    + [Components/Containers](#components-containers)
    + [Higher Order Components (HoC) vs. Render Props](#higher-order-components--hoc--vs-render-props)
  * [Testing](#testing)
    + [Unit testing](#unit-testing)
    + [End-to-end testing](#end-to-end-testing)
  * [React Hooks & Class components](#react-hooks---class-components)
  * [Tooling](#tooling)
    + [Browser Developer Tools](#browser-developer-tools)
    + [Editor Extensions](#editor-extensions)
- [TL;DR](#tl-dr)


# What is this?

Here you will find a description of various practices that you can utilize to build highly scalable, maintainable, and overall enterprise-ready React applications.

These practices will assist you in keeping your code base modular and greatly facilitate in the refactoring of your code. Moreover, it will grant you access to some amazing debugging tools and functionality such as time travel debugging and the history of all critical state changes throughout your project.

# Why?

Some of these practices are probably **NOT** for you _if_:

- You are not proficient in using your IDE of choice, or it does not have proper support for tooling (autocompletion, snippets, etc.)
- You work on a small hobby project entirely by yourself
- The project is very short-lived
- Do not care enough about your code to continuously refactor it
- Do not have a lot of critical business logic to keep track of

Obviously the more of these apply to you and your project, the worse the potential upsides of the following practices gets.

On the flip side, if you are working on a longer-lived project, possibly with multiple team members, most of these practices will greatly boost your productivity and overall output in the long haul, if properly implemented. Yes, it will lead to more infrastructure and up-front thought and time investment, and thus a slight drop in initial productivity. However, this continued effort will ensure that the complexity of your project does not grow more than linearly (which obviously drops productivity - or increases effort). The below graph (taken from [Martin Fowler](https://martinfowler.com/articles/is-quality-worth-cost.html)) visualizes this relationship well:

![graph of cumulative functionality](https://github.com/4lgn/dots-tracker/blob/master/images/graph.png "Graph of cumulative functionality")

If you need more in-depth arguments in regards to this, you can read Martin Fowler's article on _"Is High Quality Software Worth the Cost?"_ [here](https://martinfowler.com/articles/is-quality-worth-cost.html).



# Practices


## Typescript

Using TypeScript in your React project definitely has its benefits and drawbacks. However, the less of the non-applicability parameters explained in the _"Why?"_ section of this document apply to you and your project at hand, the more benefits you will experience. A pros and cons list might be appropriate in this case, so you can make up your own mind.

### Pros/cons

**Pros:**

- Huge workflow / tooling improvements:
    - Type hinting
    - Auto-completion
    - Auto imports
- Ease of collaboration between developers
- Components are more often self-explanatory by just reading required/optional props and types
    - This applies both to internal components shared between developers, but also from 3rd party libraries
- API calls and data flow are explicitly typed - leading to less time looking at documentation and/or asking co-workers
- Much easier refactoring
- Higher code confidence (more focus on integration testing instead of simple unit tests)

**Cons:**

- More code needs to be written
- Types of advanced/generic components might become hard to comprehend
- Bad/non-existant types of smaller npm packages (smaller of a problem nowadays due to the rather widespread adoption of TS)

### Should writing more code really be classified as a drawback?

I reckon the biggest and most common drawback people bring up against TypeScript is the added overhead and _"boilerplate"_ needed to write the same code. I intentionally put scare quotes around the word _boilerplate_, because: is it really boilerplate code?:

Now, it is true that in total you will end up writing more lines of code. These extra added lines of code, however, is code that requires almost zero thought at the time of writing, as you already (hopefully) have an idea of the API and needs of the function or component you are creating. This _no-brainer_ code at the time of writing will be incredibly valuable later when you have **forgot** all about how that component worked - at this point in time you wish you had explicitly told _future you_ how it worked, as it was so easy back then, but so hard to remember now. The extra knowledge you explicitly provides the TypeScript compiler makes it essentially act as a second brain of yours, constantly reminding you of how things work, and if you've written some inconsistent logic in regards to other pieces of logic you might have written months ago. It must also be noted that these effects only get amplified given you also have co-workers - which makes the compiler act as a _shared_ second brain for all developers working on the project. The compiler is a _feature_, a _helper_, its not an obstacle - you must work _with_ it, and not _against_ it.

If you bundle this with the addition of tooling such as snippets and auto-completion (upon usage), using TypeScript can become _as fast_ or, dare I say it, _faster_ than without. Below here you glance at some attached videos of how fast writing fully typed React code can be:

![typescript workflow](https://github.com/4lgn/dots-tracker/blob/master/images/typescriptworkflow.gif "Typescript Workflow")


## Redux

Using Redux is a brilliant way of decoupling your application state and other presentational logic. It helps keep critical business logic completely separate and modular - making it possible to easily tweak, change, or replace the logic independently of most of your components.

There are many different ways and patterns of using Redux, and there are usually no single _right_ answer. One of the opinionated topics that usually pops up in conversations of ways of using Redux is: _"How much, and what, should I keep in my Redux store?"_. Some users prefer to keep every single piece of data in Redux, to maintain a fully serializable and controllable version of their application at all times. Others prefer to keep non-critical, presentational state - such as "is this dropdown currently open" - inside a component's internal state.

I'm mostly a fan of the latter, as Redux is very useful, as explained previously, for decoupling application state and presentational logic. By delegating all your component's state to your store to track and serialize, you inadverdently couple your components with your store, while also cluttering up your store state with rather trivial data. This presentational logic can be argued being non-trivial, as it might make your application useless if your dropdown doesn't get triggered properly. This can, and should, however, be tested properly, which is further facilitated by the [container/component pattern](#components-containers) covered later. Moreover, Redux also takes significantly more code and design to make work properly if it must track all your component's state, compared to using a component's internal state with React Hooks.

Thus, adhere to the Single-Responsibility Principle, by making your Redux store solely responsible for keeping critical application state safe, serialized and easily debuggable. Do this by keeping **component state for component state**, and **Redux for application state**. So, if your component:

- Does not use the network
- Does not save or load state
- Does not share state with other non-child components
- Does need some ephemeral local component state

...then you should probably just use React's built-in component state model; possibly using React Hooks.


### Handling asynchronous actions


You probably already know that, unfortunately, not all state and data in web development is perfectly synchronous - it is actually mostly asynchronous. Asynchronous data fetching scattered around everywhere at every level of components, is what often leads to hard-to-debug bugs as it becomes increasingly difficult to get a proper overview of the data flow of your application. Usually this data is also critical business logic tied to your application, which makes it the perfect candidate for data to keep serialized and managed in your Redux store. Redux by itself, however, uses reducers that are strictly meant for synchronous actions. Hence, you need to use some sort of middleware to properly be able to implement an asynchronous data flow for your Redux store.

Usually, you will see two popular middleware libraries for handling asynchronous actions, namely [Redux Thunk](https://github.com/reduxjs/redux-thunk) and [Redux Saga](https://redux-saga.js.org/). There are some differences to take into account, however. Redux Saga is a bigger and more fully-fledged library that has support for handling very complex asynchronous data flows, whereas Redux Thunk more of a simple dispatcher wrapper, that turns your actions into promises. Another benefit of Redux Saga is that you can avoid callback hell, meaning that you can avoid passing in functions and calling them inside - this is due to Redux Saga using ES6's new generator functions. Additionally, you can more easily test your asynchronous data flow. The _call_ and _put_ methods return JavaScript objects. Thus, you can simply test each value yielded by your saga function with an equality comparison.

For rather simple asynchronous data flow - for example fetching some data - Redux Thunk will suffice, and should be used. However, feel free to use Redux Saga for the aforementioned benefits, just try not to overcomplicate an otherwise simple application, simply because you want to use Redux Saga.

To add to this, you might also utilize my _very opinionated_ Redux library to manage your asynchronous data flow, which reduces the usual boilerplate by an incredible amount, and essentially strips away as much _"Redux"_ thinking as possible - the only thing, almost, you need to give it is the asynchronous data fetching logic, everything else it generated for you. It is called redux-flow, check it out [here](https://www.npmjs.com/package/redux-flow).

## Project structure

### Components/Containers

WIP


## Testing

### Unit testing

WIP

### End-to-end testing

WIP

## React Hooks & Class components

WIP

## Tooling

### Browser Developer Tools

- [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)

### Editor Extensions

- [EditorConfig](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)
- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)
- [vscode-styled-components](https://marketplace.visualstudio.com/items?itemName=jpoissonnier.vscode-styled-components)


# TL;DR

First of all, this is most likely **not applicable** to _small_ and _short-sighted_ projects being _developed by 1 person_ - as you won't be able to properly reap the benefits.

Okay, now **do**:

- Use TypeScript diligently
- Leverage the power of Redux where appropriate
    - Have a standardized way of dealing with asynchronous actions
    - Design your state carefully by making it modular and composable
- Test your code
- Use React Hooks (no need for class components)
- Compose and DRY-up your codebase with Render Props or HoC's
- Constantly refactor away from containers to components wherever applicable
- Properly use IDE tooling to optimize your workflow


