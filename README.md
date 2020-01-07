# Creating a stencil library that can then be used by angular and react projects

This is a step by step guide in 

## Create a component-library:

### Cloning the component starter

```
git clone https://github.com/ionic-team/stencil-component-starter.git component-library
cd component-library
```

Modify the following files:

 - package.json -> replace `stencil-starter-project-name` with `component-library`
 - stencil.config.ts -> replace `stencil-starter-project-name` with `component-library`
 - src/index.html -> replace `stencil-starter-project-name` with `component-library`

And run

```
npm install
npm start
```

Check that everything is ok


### Preparing the bindings

In the component library add the following dependencies:

```
npm install ---save-dev @stencil/angular-output-target
npm install ---save-dev @stencil/react-output-target
```

Modify the `stencil.config.ts`. The resulting file should be

```
import { Config } from '@stencil/core';
import { angularOutputTarget, ValueAccessorConfig } from '@stencil/angular-output-target';
import { reactOutputTarget } from '@stencil/react-output-target';

const angularValueAccessorBindings: ValueAccessorConfig[] = [];

export const config: Config = {
  namespace: 'component-library',
  outputTargets: [
    angularOutputTarget({
      componentCorePackage: 'component-library',
      directivesProxyFile: '../component-library-angular/src/directives/proxies.ts',
      valueAccessorConfigs: angularValueAccessorBindings
    }),
    reactOutputTarget({
      componentCorePackage: 'component-library',
      proxiesFile: '../component-library-react/src/components.ts'
    }),
    {
      type: 'dist',
      esmLoaderPath: '../loader'
    },
    {
      type: 'docs-readme'
    },
    {
      type: 'www',
      serviceWorker: null // disable service workers
    }
  ]
};
```

## Create the component-library-angular and component-library-react libraries

We clone the templates:

```
cd .. #go back in the main directory
git clone https://github.com/ionic-team/stencil-ds-angular-template.git component-library-angular
git clone https://github.com/ionic-team/stencil-ds-react-template.git component-library-react
cd component-library-angular
cd ../component-library-react
cd ..
```

In theory, all the project names and dependencies should be aligned to `component-library`

## Build the libraries

### Stencil library

First build the component-library and pack it (this will let us circumvent the need of a npm repository and it's simpler than linking projects as it cause issues on windows)

```
cd component-library
npm run build
npm pack
cd ..
```

This will create a `component-library-0.0.1.tgz` file.

### Angular library

Modify and build the angular library

```
cd component-library-angular
npm install ../component-library/component-library-0.0.1.tgz
npm install
```

Load the correct component:

modify the file `src/component-library-module.ts`. Change 

```
import { DemoComponent } from "./directives/proxies";
```

into 

```
import { MyComponent } from "./directives/proxies";
```

And 

```
const DECLARATIONS = [
  // proxies
  DemoComponent
];
```

into

```
const DECLARATIONS = [
  // proxies
  MyComponent
];
```

Build the angular library and pack it:

```
npm run build
npm pack
```

This will create a `component-library-angular-0.0.1.tgz` file.


### React library

```
cd ..
cd component-library-react
```

First, modify the `package.json` file, as we have the following issue: https://github.com/microsoft/TypeScript/issues/32333 .

Update from:

```
"@types/node": "10.12.9"
```

to:

```
"@types/node": "13.1.4"
```

First install the dependencies

```
npm install ../component-library/component-library-0.0.1.tgz
npm install
```

And then build (note, we avoid npm run build as it will not work on windows) and pack

```
npm run compile
npm pack
```

This will create the file: `component-library-react-0.0.1.tgz`

## Creating the angular and react projects and use the component-library!


```
cd ..
```

### Angular

Create a new angular project

```
npx ng new angular-test
```

Use default option (no route, css style).

```
cd angular-test
```

Add the dependency:

```
npm install ../component-library-angular/component-library-angular-0.0.1.tgz ../component-library/component-library-0.0.1.tgz
```

Modify the file: `src/app/app.module.ts`

And import our module (`ComponentLibraryModule`), the resulting file will be:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { ComponentLibraryModule } from 'component-library-angular';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    ComponentLibraryModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Adding the component, modify src/app/app.component.html so the resulting file will simply be:

```
<my-component [first]="'From Angular'" [last]="'to Stencil'"></my-component>
```

As you can see, the @Input first and last works as expected!

Then run it:

```
npm run start
```

And enjoy :)

### React

```
cd ..
```

Create a new react project:

```
npx create-react-app react-test
cd react-test
npm install ../component-library-react/component-library-react-0.0.1.tgz ../component-library/component-library-0.0.1.tgz
```

Go into `src/App.js`

And modify it:

```
import React from 'react';
import { MyComponent } from 'component-library-react/dist/'
import './App.css';

function App() {
  return (
    <div className="App">
      <MyComponent first='From React' last='to Stencil'></MyComponent>
    </div>
  );
}

export default App;
```

And then run it

```
npm start
```

And enjoy!