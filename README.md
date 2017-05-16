# Angular dojo
It's time to do some Angular! This dojo contains a bunch of things that Matt likes about Angular, and only scratches the surface of what it is capable of. If you're interested in learning more, the best place to start is the official docs: [https://angular.io/docs/ts/latest/](https://angular.io/docs/ts/latest/)

## Pre-requisites
* Install [node](https://nodejs.org/en/download/) & [npm](https://www.npmjs.com/). You can use [yarn](https://yarnpkg.com/lang/en/) if you prefer!
* Install [git](https://git-for-windows.github.io/)
* Install angular-cli: `npm install -g @angular/cli`
* Install gulp: `npm install -g gulp`
* Install [vs2017](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15#)
* Install [vscode](https://code.visualstudio.com/), and this [auto import extension](https://marketplace.visualstudio.com/items?itemName=steoates.autoimport)

## Project set-up
Follow these steps to create a blank project, and run it locally. Keep it running for the rest of this dojo:

* Open a new git bash window (or cmd if you prefer)
* Change directory to where you'd like to create your new project folder
* `ng new angular-dojo-frontend`
* `cd angular-dojo-frontend`
* `code .` (or open your favourite text editor and browse to your new project)
* `ng serve`

You should now be able to browse to your shiny new Angular site, using the port shown in your command window. (i.e. http://localhost:4200)

### Styling
For this dojo, we'll use [Semantic UI](https://semantic-ui.com/) as our CSS framework. First we need to install Semantic UI, and build the CSS and JS:

* Open another git bash window (or cmd if you prefer). We'll use two windows, so you can leave your other one serving your app.
* `npm install semantic-ui jquery --save` (Note: due to a quirk in the semantic setup, you may need to run this in cmd rather than git bash. Install using all the defaults)
* `cd semantic`
* `gulp build`

Now we need to reference the compiled semantic CSS and JS, along with jquery. Crack open vscode, and edit `angular-cli.json`:

```
"apps": [{
  ... 
  "styles": [
      "styles.css",
      "../semantic/dist/semantic.min.css"
  ],
  "scripts": [
      "../node_modules/jquery/dist/jquery.min.js",
      "../semantic/dist/semantic.min.js"
  ],
  ...
}]
```

You'll need to kill and re-run `ng serve` in the first command window, so that it picks up the updated Angular config. Now let's see if it's working, by editing `app.component.html`:

* Wrap the contents of the component in a `div`, with class `ui container segment`
* Edit the `h1`, so it has class `ui header`
* Add a `p` below the `h1`, with a bit of content

If you've left the Angular site running, you should see all of your changes show up in real time. For **bonus points** - read up on view encapsulation and shadow DOM.

## Binding
Let's play around with some Angular binding, by editing `app.component.html`:

* Add `<input [(ngModel)]="title">` to the component
* Wrap the input in a `<div>` with class `ui input`

Note that this is a **two way binding**. Notice how the value displayed in the `<h1>` (which is using a **one way binding**) automatically changes in real time, as you edit the input field.

## Components
Create a new navigation component using angular-cli:

* cd [wherever-you-installed-your-app]/angular-dojo-frontend
* `ng g component navigation`

Notice how it creates the scaffolding for our new component, and also edits our app definition in `app.module.ts`. Now let's add it into our app:

* Edit `/navigation/navigation.component.html`. Add a semantic menu, something like:
```
<div class="ui top menu">
  <a class="item">Home</a>
  <a class="item">About</a>
</div>
```
* Add `<app-navigation></app-navigation>` to the top of `app.component.html`.

## Routing
Let's get some routing going:

* Create two new components:
  * home
  * about 
* Prettify them, by wrapping the contents of their HTML in a `<div class="ui container segment">`
* Add a `<router-outlet></router-outlet>` to the bottom of `app.component.html`.
* Add an `app.routing.ts` file to the app root, with contents like:
```
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent }
]

export const routing = RouterModule.forRoot(routes);
```
* Edit `app.module.ts`, and add `RouterModule` and `routing` to the `imports` list
* Edit `navigation.component.html`, and add attributes `[routerLink]="['']"` and `[routerLink]="['about']"` to their respective `<a>`s
* Bask in your own glory. For bonus points - look into how Angular handles **child routes**

## Services
Let's create a service, so that we can put some data on the screen:

* `ng g service data-access`
* Edit the new service, and give it a method that returns some values. Something like:
```
getValues(): string[] {
  return ["1", "2"];
}
```
* **Inject** the new service into `about.component.ts`. **Don't** instantiate it! (**Hint:** `constructor(private dataAccessService: DataAccessService)`)
* **Provide** the new service in `about.component.ts`. **Hint:** `providers: [DataAccessService]`
* Call the new `getValues()` method when the `about` component initiates (the lifecycle hook should already be there, thanks to angular-cli). Assign the return value to a local variable (say `values`)
* Edit `about.component.html`, to actually display the values. Here we can use some Angular syntax, like:
```
<ul>
  <li *ngFor="let value of values">
    {{value}}
  </li>
</ul>
```

### Asynchronicity
To simulate real-life, let's make our service **asynchronous**, and introduce an artificial wait-time into our service. Start by editing `data-access.service.ts`:

* Change the signature of our `getValues()` method, so that it returns a `Observable<string[]>`
* Change the return value to `Observable.of(["1", "2"])`

We'll now have a compile error. Fix this by editing `about.component.ts`:

* In `ngOnInit`, change the line that calls the service to: `this.dataAccessService.getValues().subscribe(result => this.values = result);`

We're not quite there, as our service is still returning data instantaneously. Let's slow it down, by editing the return value of `getValues()` in `data-access.service.ts` again:

```
return new Observable(subscriber => {
  setTimeout(() => subscriber.next(["1", "2"]), 1000);
});
```

Now test it out, and *observe* how the about component takes a second to display the values on the screen, every time it is initiated. We can take this one step further, and display something on screen whilst we're waiting. Let's do that now:

* In `about.component.ts`, add a new local variable, `isLoading: boolean`
* Also in `about.component.ts`, inside `ngOnInit`, start by setting `this.isLoading = true`. When the values are returned from the service, set `this.isLoading = false`
* In `about.component.html`, display something when loading:
```
<div *ngIf="isLoading" class="ui active centered inline text loader">
  Loading...
</div>
```

### Route resolvers
In real life you might not want to instantly load the page and show some holding content whilst waiting for data. You may want to wait until all the data is present before activating the route. This can be achieved with **route resolvers**. They're not covered in this dojo, but are fairly straightforward. You can read more at [thoughtram](https://blog.thoughtram.io/angular/2016/10/10/resolving-route-data-in-angular-2.html).

## Interfacing with an API
This repository contains a pre-built dotnet core webapi project. You *could* clone this repo, then open the solution within `./api/src/` in Visual Studio, and run the webapi project. Or, if you're lazy, you can use the pre-existing backend API in Azure, at: http://angular-dojo.azurewebsites.net

### Swagger
With the webapi project running, you should be able to browse to the Swagger endpoints, thanks to [NSwag](https://github.com/NSwag/NSwag):

* Swagger UI: http://angular-dojo.azurewebsites.net/swagger (or http://localhost:4201/swagger if you're running locally)
* Swagger JSON: http://angular-dojo.azurewebsites.net/swagger/v1/swagger.json (or http://localhost:4201/swagger/v1/swagger.json if you're running locally)

### NSwag client
Let's use NSwag to generate a TypeScript client for us. Back in our dojo front-end project:

* Install NSwag 8.0.0*: `npm install nswag@8.0.0 -g`
* Generate the TS client: `nswag swagger2tsclient /input:http://angular-dojo.azurewebsites.net/swagger/v1/swagger.json /output:./src/api/apiclient.ts /template:angular2` (you might want to add this command as a script inside `package.json`)
* Edit `about.component.ts`:
  * Add import: `import * as apiClient from '../../api/apiclient';`
  * Add provider: `providers: [apiClient.ValuesClient]`
  * Change injected service: `valuesClient: apiClient.ValuesClient`
  * Change method call: `this.valuesClient.getAll().subscribe(result => ...`

Check that the data shows up on the screen.

*For some reason the latest version of NSwag wasn't working properly at the time of writing, but 8.0.0 was. Feel free to try the latest version if you want. Note that the template name is simply `angular`, not `angular2` when using the latest version.

## Forms, posting and waiting
Let's create a quick form:

* Create a new component called `form`
* Add a new route for this new component, in `app.routing.ts`
* Add a new `routerLink` into our nav component, so we can navigate to it
* Edit the new `form.component.ts`:
  * Add a local variable `myInput: string`
  * Create a blank method `submit()`. We'll use this later
  * Add `import * as apiClient from '../../api/apiclient';`
  * Inject `apiClient.ValuesClient` into the constructor, as before
* Move the line that provides `apiClient.ValuesClient`. This is currently in `about.component.ts`. We can move this up a layer, so that it is provided in `app.module.ts`, to save us having to duplicate any code
* Edit the new `form.component.html`:
  * Replace the default contents with a `<form>` with class `ui form`. Also add `(ngSubmit)="submit()"` to this `<form>`. This hooks the submit method into the `submit()` method we defined earlier on the component
  * In the form, add an `<input>` of type `text`, with `[(ngModel)]` set to the local variable `myInput` created earlier. You'll also need to set the `name` to something, otherwise Angular will moan
  * Wrap this `<input>` in a `<div>` with class `field`
  * Add a `<button>` of type `submit`, with class `ui button green`
  * Wrap the contents of the `<form>` in a `<div>` with class `ui container segment`

Now, because the form input is bound to our local variable `myInput`, when we submit the form we can access the user's input through this variable. This lets us easily post to the api from inside the component. Edit `form.component.ts`:

* Inside `submit()`, add a line like `this.valuesClient.post(this.myInput).subscribe();`

If you put a breakpoint in Visual Studio, inside the post method of `ValuesController`, you should see that your entered value has arrived on the server. Cool!

However, it could be cooler by being a bit more responsive. We can change that pretty easily:

* Add a new local variable to `form.component.ts`: `isSubmitting: boolean;`
* In `submit()`, start by setting `this.isSubmitting = true`. When the response comes back from the api, set this back to `this.isSubmitting = false`
* In `form.component.html`, add `[disabled]="isSubmitting"` to both the `<input>` and `<button>`
* Add `[class.loading]="isSubmitting"` to the `<button>`

Try submitting the form again. Cooler!

**Bonus points:** You can move the `[disabled]="isSubmitting"` attribute onto a `<fieldset>` wrapping all the form elements, rather than replicating it on each element.

## Pipes
Let's check out Angular's pipes:

* Edit `home.component.ts` - add a local variable `exampleDate: Date = new Date();`
* Edit `home.component.html`- display `exampleDate`, using `{{exampleDate}}`

Notice how the full Date object is displayed. Let's change that with a pipe:

* Change the binding to `{{ exampleDate | date }}`

Ooh shiny. You can of course create your own pipes, and you can parameterise them too.

## Attribute directives
Let's check out Angular's attribute directives. Create a new directive using angular-cli:
```
ng g directive highlight
```

Now edit  `highlight.directive.ts`:

* Inject `private el: ElementRef` into the constructor
* Make the directive implement `OnInit`, and add the `ngOnInit` method. Edit it to something like:
```
ngOnInit() {
  this.el.nativeElement.classList.add("ui", "segment", "yellow");
}
```

Now let's actually use our directive, by adding it as an attribute to a div:
```
<div appHighlight>
```

Hooray! We can also add `@Input` properties onto directives, for further customisability. Edit `highlight.directive.ts`:

* Add an input property: `@Input() highlightColor: string;`
* Use this new property, instead of hard-coding `"yellow"`, in `ngOnInit()`

Now we can use our directive along with its input property:
```
<div appHighlight highlightColor="yellow">
```

**Bonus points:** make the syntax more compact, so that we can just write `<div appHighlight="yellow">`. You can do this by matching the name of the `@Input` property to the directive's name. To avoid confusion inside the directive, you can use an **input alias**.

## Unit tests
You may have noticed that angular-cli has been generating `*.spec.ts` files whenever we create anything. And in fact, they come with a few built-in tests, ready to go. To run them, use:
```
ng test
```

Woops - they fail! We need to do some quick fixes to accommodate all the code we've already written:

* In `about.component.spec.ts`, we need to import Angular's HTTP module. Add `imports: [HttpModule]` to the `TestBed.configureTestingModule` method
* Also in `about.component.spect.ts`, we need to provide the ValuesClient. Add `providers: [apiClient.ValuesClient]` (and `import * as apiClient from '../../api/apiclient';`)
* In `app.component.spec.ts`, we need to tell Angular about our custom components. Add `schemas: [CUSTOM_ELEMENTS_SCHEMA]` (and import `CUSTOM_ELEMENTS_SCHEMA` from `@angular/core`)
* Also in `app.component.spec.ts`, we need to import Angular's forms module. Add `imports: [FormsModule]`
* In `navigation.component.spec.ts`, we need to import Angular's router testing module. Add `imports: [RouterTestingModule]` (and import `RouterTestingModule` from `@angular/router/testing`)
* In `form.component.spec.ts`, we need to import both Angular's HTTP module, Angular's forms module, and we need to provide the ValuesClient. Use the same approach as above
* Finally, just delete `highlight.directive.spec.ts`, as we're not going to test it in this dojo

And now, after running `ng test` again, the tests should all pass.. hooray!

### Mocking our service
Let's mock our `apiClient.ValuesClient`, so we can test that the about component is populating itself correctly. Edit `about.component.spec.ts`:

* Create a class, `MockValuesClient` that extends `apiClient.ValuesClient`. Override `getAll()`, by defining a mock method that returns `Observable.of(["value3"]);`
* Configure the test bed to use an instance of `MockValuesClient` whenever `apiClient.ValuesClient` is needed. Here's where providers come in handy, as we can change out the providers line to: `providers: [{ provide: apiClient.ValuesClient, useClass: MockValuesClient }]`
* Create a new test method to check that the values from the api are populating the component's local variable. Something like `expect(component.values).toContain("value3");`

The test should pass. **Bonus points:** take it one step further, and test that the value from the mock client is displayed on the screen.

## Debugging and building
Angular CLI runs on [webpack](https://webpack.github.io/). When running the local webpack dev server (i.e. `ng serve`), the best way to debug your code is through Chrome:

* Open the dev console (F12)
* Go to Sources
* Ctrl + P -> start typing the name of the .ts file you want to debug
* Add a breakpoint

To build the solution ready for deployment, just run `ng build` (or `ng build --prod` if you want to remove source maps, and run uglify). You can host the `/dist` folder on any web server.

## Useful tools

* [Augury](https://chrome.google.com/webstore/detail/augury/elgalmkoelokbchhkhacckoklkejnhcd?hl=en)
