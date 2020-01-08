# Creating a stencil library that can then be used by angular and react projects

This is a step by step guide based from the source code of https://github.com/ionic-team/stencil-ds-plugins-demo.

I hope it show a little bit more the underlying machinery/requirements.

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
rm package-lock.json
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

## Adding a new component

In this section we will explore what entails adding a new "input type" component
as it has some non trivial integration issues with Angular.

Go to `component-library`.

Run:

```
npx stencil generate my-custom-input
```

Press enter.

It will create a directory src/components/custom-input.

Now, modify the custom-input.tsx file as the following:

```
import { Component, Event, h, Prop, EventEmitter } from '@stencil/core';

@Component({
  tag: 'my-custom-input',
  styleUrl: 'custom-input.css',
  shadow: true
})
export class CustomInput {

  @Prop({ mutable: true }) value?: string | null = "";

  @Event()
  myChange: EventEmitter;

  changeOccurred = (e: Event) => {
    this.value = (e.currentTarget as HTMLInputElement).value;
    this.myChange.emit(this.value);
  };

  render() {
    return (
      <div>
        <label>My Custom Input</label>
        <input type="text" value={this.value} onChange={this.changeOccurred}></input>
      </div>
    );
  }

}
```

and add the tag:

```
<my-custom-input value="Hello world!"></my-custom-input>
```

in src/index.html.

run

```
npm start
```

and check that your custom input is available.

Now build again:

```
npm run build
npm pack
```

### Updating the angular binding

Go to the component-library-angular directory and install our newly packed component library:

```
cd ..
cd component-library-angular
npm install ../component-library/component-library-0.0.1.tgz
```

As you can see the `src/directives/proxies.ts` file is now updated with our new component!

The next step will be to integrate our `my-custom-input` element with the ngModel/reactive form machinery of Angular.

For this we need to write a custom `ControlValueAccessor` (see https://github.com/ionic-team/stencil-ds-plugins-demo/blob/master/packages/component-library-angular/src/directives/value-accessor.ts)

Add a new file: `src/directives/value-accessor.ts` with the following content:

```
import { ElementRef, HostListener } from '@angular/core';
import { ControlValueAccessor } from '@angular/forms';

export class ValueAccessor implements ControlValueAccessor {

  private onChange: (value: any) => void = () => {/**/};
  private onTouched: () => void = () => {/**/};
  protected lastValue: any;

  constructor(protected el: ElementRef) {}

  writeValue(value: any) {
    this.el.nativeElement.value = this.lastValue = value == null ? '' : value;
  }

  handleChangeEvent(value: any) {
    if (value !== this.lastValue) {
      this.lastValue = value;
      this.onChange(value);
    }
  }

  @HostListener('focusout')
  _handleBlurEvent() {
    this.onTouched();
  }

  registerOnChange(fn: (value: any) => void) {
    this.onChange = fn;
  }
  registerOnTouched(fn: () => void) {
    this.onTouched = fn;
  }
}
```

And for the concrete "binding" to our component, add a new file `src/directives/my-custom-input-value-accessor.ts` with the following content:

```
import { Directive, ElementRef } from '@angular/core';
import { NG_VALUE_ACCESSOR } from '@angular/forms';

import { ValueAccessor } from './value-accessor';

@Directive({
  /* tslint:disable-next-line:directive-selector */
  selector: 'my-custom-input',
  host: {
    '(myChange)': 'handleChangeEvent($event.target.value)'
  },
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: MyCustomInputValueAccessor,
      multi: true
    }
  ]
})
export class MyCustomInputValueAccessor extends ValueAccessor {
  constructor(el: ElementRef) {
    super(el);
  }
  registerOnChange(fn: (_: number | null) => void) {
    super.registerOnChange(value => {
      fn(value);
    });
  }
}
```

Finally you need to add the new component and the value accessor in `src/component-library-module.ts`:

```
import { NgModule } from "@angular/core";
import { defineCustomElements } from "component-library/loader";

import { MyComponent, MyCustomInput } from "./directives/proxies";
import { MyCustomInputValueAccessor } from "./directives/my-custom-input-value-accessor"

defineCustomElements(window);

const DECLARATIONS = [
  // proxies
  MyComponent,
  MyCustomInput,

  // value accessor
  MyCustomInputValueAccessor
];

@NgModule({
  declarations: DECLARATIONS,
  exports: DECLARATIONS,
  imports: [],
  providers: []
})
export class ComponentLibraryModule {}

```

and finally build and pack:

```
npm run build
npm pack
```

### Testing the angular binding

Go in the angular project 

```
cd ..
cd angular-test
```

And update the dependencies:

```
npm install ../component-library-angular/component-library-angular-0.0.1.tgz ../component-library/component-library-0.0.1.tgz
```

Modify the `src/app/app.module.ts` to add the `FormModule`:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { ComponentLibraryModule } from 'component-library-angular';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    ComponentLibraryModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Change `src/app/app.component.html`:

```
<my-component [first]="'From Angular'" [last]="'to Stencil'"></my-component>

<my-custom-input (myChange)="customInputChanged($event)" [(ngModel)]="customInputValue"></my-custom-input>

{{customInputValue}}
```

And `src/app/app.component.ts`

```
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'angular-test';

  customInputValue = 'initial value'

  customInputChanged(event) {
    console.log(event);
  }
}
```


## Note about custom components/angular binding

You may notice something: the events must be custom ones, e.g. `change` becomes `myChange`. When building custom elements, always
think about what should be exposed in term of events: blur? focus? change?

For angular components, when you are creating "input" type of elements, you must implement manually a `ValueAccessor`. This may be tedious,
but it only need to be done once.
