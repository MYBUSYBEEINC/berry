---
category: features
slug: /features/constraints
title: "Constraints"
description: An in-depth guide to Yarn's constraints, a feature that provides an easy way to enforce common rules across a project.
---

:::info
This page documents the new JavaScript-based constraints. The older constraints, based on Prolog, are still supported but should be considered deprecated. Their documentation can be found [here](/features/constraints-prolog).
:::

## Overview

Constraints are a solution to a very basic need: you have a lot of [workspaces](/features/workspaces), and you need to make sure they use the same version of their dependencies. Or that they don't depend on a specific package. Or that they use a specific type of dependency. Whatever is the exact logic, your goal is the same: you want to automatically enforce some kind of rule across all your workspaces. That's exactly what constraints are for.

## What can we enforce?

Our constraint engine currently supports two main targets:

- Workspace dependencies
- Arbitrary package.json fields

It currently doesn't support the following, but might in the future (PRs welcome!):

- Transitive dependencies
- Project structure

## Creating a constraint

Constraints are created by adding a `yarn.config.cjs` file at the root of your project (repository). This file should export an object with a `constraints` method. This method will be called by the constraints engine, and must define the rules to enforce on the project, using the provided API.

For example, the following `yarn.config.cjs` will enforce that all `react` dependencies are set to `18.0.0`.

```ts
module.exports = {
  async constraints({Yarn}) {
    for (const dep of Yarn.dependencies({ ident: 'react' })) {
      dep.update(`18.0.0`);
    }
  },
};
```

And the following will enforce that the `engines.node` field is properly set in all workspaces:

```ts
module.exports = {
  async constraints({Yarn}) {
    for (const workspace of Yarn.workspaces()) {
      workspace.set('engines.node', `20.0.0`);
    }
  },
};
```

## Declarative model

As much as possible, constraints are defined using a declarative model: you declare what the expected state should be, and Yarn checks whether it matches the reality or not. If it doesn't, Yarn will either throw an error (when calling `yarn constraints` without arguments), or attempt to automatically fix the issue (when calling `yarn constraints --fix`).

Because of this declarative model, you don't need to check the actual values yourself. For instance, the `if` condition here is extraneous and should be removed:

```ts
module.exports = {
  async constraints({Yarn}) {
    for (const dep of Yarn.dependencies({ ident: 'ts-node' })) {
      // No need to check for the actual value! Just always call `update`.
      if (dep.range !== `18.0.0`) {
        dep.update(`18.0.0`);
      }
    }
  },
};
```

## TypeScript support

Yarn ships types that make it easier to write constraints. To use them, add the dependency to your project:

```
$ yarn add @yarnpkg/types
```

Then, in your `yarn.config.cjs` file, import the types, in particular the `defineConfig` function which automatically type the configuration methods:

```ts
/** @type {import('@yarnpkg/types')} */
const { defineConfig } = require('@yarnpkg/types');

module.exports = defineConfig({
  async constraints({Yarn}) {
    // `Yarn` is now well-typed ✨
  },
});
```

You can also retrieve the types manually, which can be useful if you extract some rules into helper functions:

```ts
/** @param {import('@yarnpkg/types').Yarn.Constraints.Workspace} dependency */
function expectMyCustomRule(dependency) {
  // ...
}
```

You can alias the types to make them a little easier to use:

```ts
/**
 * @typedef {import('@yarnpkg/types').Yarn.Constraints.Workspace} Workspace
 * @typedef {import('@yarnpkg/types').Yarn.Constraints.Dependency} Dependency
 */

/** @param {Workspace} dependency */
function expectMyCustomRule(dependency) {
  // ...
}
```
