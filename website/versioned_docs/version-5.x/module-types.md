---
id: module-types
title: single-spa Microfrontend Types
sidebar_label: Microfrontend Types
---

# Concept: single-spa Microfrontend Types

Single-spa has [different categories](./microfrontends-concept#types-of-microfrontends.md) of microfrontends. It is up to you where and how you use each of them. However, the single-spa core team has [recommendations](./recommended-setup/#applications-versus-parcels-versus-utility-modules.md).

Here is how each single-spa microfrontend works conceptually. This information should help you understand our [recommendations](./recommended-setup/#applications-versus-parcels-versus-utility-modules.md).

| Topic                | Application                       | Parcel                               | Utility                              |
| -------------------- | --------------------------------- | ------------------------------------ | ------------------------------------ |
| Routing              | has multiple routes               | has no routes                        | has no routes                        |
| API                  | declarative API                   | imperative API                       | no single-spa API                    |
| Renders UI           | renders UI                        | renders UI                           | doesn't directly render UI           |
| Lifecycles           | single-spa managed lifecycles     | custom managed lifecycles            | no lifecycles                        |
| When to use          | core building block               | only needed with multiple frameworks | useful to share common logic         |

Each single-spa microfrontend is an in-browser JavaScript module ([explanation](./recommended-setup#in-browser-versus-build-time-modules.md)).

## Applications

### Applications are declarative
Applications use a declarative API called `registerApplication`. Your single-spa config (also sometimes called the root config) defines applications ahead of time and defines the conditions for when each application is active, but it doesn't mount the applications directly.

### Applications have managed lifecycles
single-spa manages registered applications and is in charge of all of their lifecycles. This prevents you from writing a bunch of logic about when applications should mount and unmount; single-spa takes care of that for you.
All that single-spa needs to make this work automatically is an activity function that describes when your application should be active.

## Parcels

### Parcels are imperative
Parcels exist in many ways as an escape hatch from the normal declarative flow. They exist primarily to allow you to reuse pieces of UI across applications when those applications are written in multiple frameworks.

### You manage the lifecycles of parcels
When you call `mountParcel` or `mountRootParcel` [(see API)](./parcels-api.md) the parcel is mounted immediately and returns the parcel object. You need to call the `unmount` method on the parcel manually when the component that calls `mountParcel` unmounts.

### Parcels are best suited for sharing pieces of UI between frameworks
Creating a parcel is as easy as using the [single-spa helpers](./ecosystem#help-for-frameworks.md) for that framework on a specific component/UI. This returns an object (`parcelConfig`) that single-spa can use to create and mount a parcel.
Because single-spa can mount a parcel anywhere, this gives you a way to share UI/components across frameworks. It should not be used if the shared UI is being used in another application of the same framework.
For example: `application1` is written in Vue and contains all the UI and logic to create a user. `application2` is written in React and needs to create a user. Using a single-spa parcel allows you to wrap your `application1` Vue component
in a way that will make it work inside `application2` despite the different frameworks.
Think of parcels as a single-spa specific implementation of webcomponents.

## Utilities

### Utility modules share common logic
Utility modules are a great place to share common logic. Instead of each application creating their own implementation of common logic, you can use a plain JavaScript object (single-spa utility) to share that logic.
For example: Authorization. How does each application know which user is logged in? You could have each application ask the server or read a JWT but that creates duplicate work in each application.
Using the utility module pattern would allow you to create one module that implements the authorization logic. This module would export any needed methods, and then your other single-spa applications could use those authorization methods by importing them.
This approach also works well for data [fetching](./recommended-setup#api-data.md).
