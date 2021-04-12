# Feature Toggle

## What's Feature Toggle

A feature toggle (also feature switch, feature flag, feature gate, feature flipper, conditional feature, etc.) is a technique in software development that attempts to provide an alternative to maintaining multiple branches in source code (known as feature branches), such that a software feature can be tested even before it is completed and ready for release. A feature toggle is used to hide, enable or disable the feature during runtime. For example, during the development process, a developer can enable the feature for testing and disable it for other users.[^1]

### Why it is good?

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

## When we do consider?

* Hiding or disabling new features in the UI
* Hiding or disabling new components in the application
* Versioning an interface
* Extending an interface
* Supporting multiple versions of a component
* Adding a new feature to an existing application
* Enhancing an existing feature in an existing application

## Toggle Configuration

This is a summary that I did for feature toggle task in an React Application.


1. Set a new environment variable `COMMENT_FEATURE_ENABLED` in yaml files.

In `integration.yaml`:

```
namespace: integration
deployment:
  containers:
    - ....
      env:
        - name: COMMENT_FEATURE_ENABLED
          value: true
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

Create `FeatureProtectedRoute` component to enable/disable router based on env.

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
  console.log(featureName, process.env);
  return (!isEnabled) ?  <></> : <Route {...routeProps} />;
};
```

In `App.tsx`

```tsx
<Route path="/" component={main.component} />
<FeatureProtectedRoute
  feature="comments"
  path="/feature-a"
  exact={true}
  component={componentA} />
```

3. Sagas, Reducers, listeningEvents..

```tsx
import { ModuleA, ModuleB, ModuleC, Comments } from './modules';

const unprotectedModules = [
  moduleA,
  moduleB,
  moduleC,
];

const protectedModules = [{
  ...Comments,
  env: process.env.COMMENT_FEATURE_ENABLED,
}];

type ModuleItem = Module & {
  env?: string;
};

interface ActiveModulesArgs {
  unprotectedModules: ModuleItem[];
  protectedModules: ModuleItem[];
  type: {
    name: 'sagas' | 'listeningEvents';
    initialValue: any;
  };
}

const getActiveModules = ({ unprotectedModules, protectedModules, type }: ActiveModulesArgs) => {
  const { name, initialValue } = type;
  const moduleFeatures = unprotectedModules.reduce((prev: any[], current) => {
    return [
      ...prev,
      ...current[name],
    ];
  }, []);

  const flaggedModuleFeatures = protectedModules.reduce((prev: any[], current) => {
    const { env } = current;
    const feature = env ? current[name] : initialValue;
    return [
      ...prev,
      ...feature,
    ];
  }, []);

  return [...moduleFeatures, ...flaggedModuleFeatures];
};

interface ActiveReducersArgs {
  unprotectedModules: ModuleItem[];
  protectedModules: ModuleItem[];
}

const getActiveReducers = ({ unprotectedModules, protectedModules }: ActiveReducersArgs) => {
  // tslint:disable-next-line: no-any
  const moduleFeatures = unprotectedModules.reduce((prev, current) => {
    return {
      ...prev,
      [current.name]: current.reducer,
    };
  }, {});

  // tslint:disable-next-line: no-any
  const flaggedModuleFeatures = protectedModules.reduce((prev, current) => {
    const { env } = current;
    return {
      ...prev,
      [current.name]: env ? current.reducer : undefined,
    };
  }, {});
  return { ...moduleFeatures, ...flaggedModuleFeatures };
};

export default {
  name: 'allModules',
  sagas: getActiveModules({ protectedModules, unprotectedModules,
    type: { name: 'sagas', initialValue: [] }}),
  listeningEvents: getActiveModules({ protectedModules, unprotectedModules,
    type: { name: 'listeningEvents', initialValue: [] }}),
  reducer: getActiveReducers({ protectedModules, unprotectedModules }),
  rootComponent: app.routeComponents.main,
};
```

### References

- [ALM Rangers : Software Development with Feature Toggles](https://docs.microsoft.com/en-us/archive/msdn-magazine/2014/may/alm-rangers-software-development-with-feature-toggles) by Bill Heys
- [Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) by Pete Hodgson
