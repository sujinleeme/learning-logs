# Feature Toggle

## What's Feature Toggle

A feature toggle (also feature switch, feature flag, feature gate, feature flipper, conditional feature, etc.) is a technique in software development that attempts to provide an alternative to maintaining multiple branches in source code (known as feature branches), such that a software feature can be tested even before it is completed and ready for release. A feature toggle is used to hide, enable or disable the feature during runtime. For example, during the development process, a developer can enable the feature for testing and disable it for other users.

## Why it is good?

Including pending or incomplete code?

* To prevent developers' work from impacting the rest of the team or destabilizing the codebase.
* Increases a team’s development speed by reducing or eliminating the need for parallel development branches and the ensuing branching and merging tasks, which can be extremely time-consuming and error-prone.
* Using feature toggles with continuous integration, you work directly in the main branch. Code is checked in to the main branch, while keeping this branch stable for builds and deployments.
* Gives runtime isolation of a feature being developed until it’s complete, tested and ready for release.

*Cautions*

* Savvy teams view their Feature Toggles as inventory which comes with a carrying cost, and work to keep that inventory as low as possible.

## Feature Toggle vs. Feature Branch

* Feature toggle: Continuous integration. All pending changes are checked into the main branch. Each check-in is pended until an automated build process builds all code from the main branch on a build server, and successfully runs automated BVTs.
* Feature Branch: All development is checked into the associated feature branch and isolated from code in the main branch or other feature branches. You can’t merge a new or enhanced feature with the main branch or other feature branches until the feature is Code Complete, passes required quality tests or meets the DoD.

## When we do consider using Feature Toggle?

* Hiding or disabling new features in the UI
* Hiding or disabling new components in the application
* Versioning an interface
* Extending an interface
* Supporting multiple versions of a component
* Adding a new feature to an existing application
* Enhancing an existing feature in an existing application

## Feature Toggle Configuration in React App

This is an example code and summary that I did for feature toggle task in a React application.

Let's assume that we want to enable `Comments` feature using a feature toggle in staging only.

Each module consists of module's `name`, `routeComponents`, `components`, `reducer`, and `sagas` properties.

```ts
type ComponentsConfig<ComponentsKeys extends string> = {
  [key in ComponentsKeys]: React.FC<any> | React.ComponentClass<any, any>;
};

export type Module<
  S,
  A extends Action<any>,
  RouteComponentKeys extends string
> = {
  name: string;
  components: ComponentsConfig<ComponentsKeys>;
  routeComponents: ComponentsConfig<RouteComponentKeys>;
  reducer?: Reducer<S, A>;
  // tslint:disable: no-any
  sagas: any[];
};
```

In the entry point of `Comments` module file, it has the below of code lines.

```javascript
import { CommentsContainer } './comments.container';

const MODULE_NAME = 'comments';

export const Comments = {
  reducer,
  routeComponents: {
    main: CommentRoutes,
  };
  components: {},
  name: MODULE_NAME,
  sagas: [],
};
```

1. Env configuration
First of all, we need to set a new environment variable, `COMMENT_FEATURE_ENABLED` in all yaml files where it set values that need to be configurable during deployment.

In `integration.yaml`:

```
namespace: integration
deployment:
  containers:
    - ....
      env:
        - name: COMMENT_FEATURE_ENABLED
          value: false
```

In `staging.yaml`:

```
namespace: integration
deployment:
  containers:
    - ....
      env:
        - name: COMMENT_FEATURE_ENABLED
          value: true
```

2. Router configuration

I created `FeatureProtectedRoute` component to enable/disable router based on env variable.

```tsx
import React from 'react';
import { Route, RouteProps } from 'react-router-dom';

const featureFlags = {
  comments : process.env.COMMENT_FEATURE_ENABLED,
};

interface FeatureProtectedRouteProps {
  feature: keyof typeof featureFlags;
}

export const FeatureProtectedRoute:  React.FC<FeatureProtectedRouteProps & RouteProps> = (
  { feature, ...routeProps }) => {
  const isEnabled = !!featureFlags[feature as keyof typeof featureFlags];
  return (!isEnabled) ?  <></> : <Route {...routeProps} />;
};
```

As like basic `<Route />`, `<FeatureProtectedRoute>` can take all props and render `<Route />` or  empty `<></>`. Only thing that we do care is `feature` prop which points environment variable name.

```tsx
import { WorkSpace } from './WorkSpace';
import { Comments } from './Comments';

<Switch>
  <Route path="/" component={WorkSpace.routeComponents.main} />
  <FeatureProtectedRoute
    feature="comments"
    path="/comments"
    exact={true}
    component={Comments.routeComponents.main} />
<Switch>
```

3. Sagas, Reducers.

Finally, we can manage reduces and sagas based on feature flags.

```tsx
import { AnyAction, Reducer } from 'redux';
import { ModuleA, ModuleB, ModuleC, Comments } from './modules';

// tslint:disable-next-line: no-any
import { Saga } from 'redux-saga';

//...

type ModuleItem = Module<any, any, any, never> & {
  env?: string;
};

enum PartKey {
  sagas = 'sagas',
}

const modules: ModuleItem[] = [
  moduleA,
  moduleB,
  moduleC,
];

const protectedModules = [
  {
    ...packagesMall,
    env: process.env.TAX_MALL_FEATURE_ENABLED,
  },
];

const getActiveModules = (
  modules: ModuleItem[],
  protectedModules: ModuleItem[],
  partKey: PartKey,
): Saga[] => {
  const moduleFeatures = modules.reduce((prev: Saga[], current: ModuleItem) => {
    if (!current[partKey]) {
      return prev;
    }
    return [...prev, ...current[partKey]];
  }, []);

  const flaggedModuleFeatures = protectedModules.reduce((prev: Saga[], current: ModuleItem) => {
    const { env } = current;
    if (!env || !current[partKey]) {
      return prev;
    }
    return [...prev, ...current[partKey]];
  }, []);

  return [...moduleFeatures, ...flaggedModuleFeatures];
};

const getActiveReducers = (
  modules: ModuleItem[],
  protectedModules: ModuleItem[],
): {
  [key: string]: Reducer;
} => {
  const moduleFeatures = modules.reduce((prev, { name, reducer }) => {
    if (!reducer) {
      return prev;
    }

    return {
      ...prev,
      [name]: reducer,
    };
  }, {});

  const flaggedModuleFeatures = protectedModules.reduce((prev, { env, name, reducer }) => {
    if (!env || !reducer) {
      return prev;
    }

    return {
      ...prev,
      [name]: reducer,
    };
  }, {});
  return { ...moduleFeatures, ...flaggedModuleFeatures };
};
```

### References

- [ALM Rangers : Software Development with Feature Toggles](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/may/alm-rangers-software-development-with-feature-toggles) by Bill Heys
- [Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) by Pete Hodgson
