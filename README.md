# Angular dojo
It's time to do some Angular! This dojo only scratches the surface of what Angular is capable of. If you're interested in learning more, the best place to start is the official docs: [https://angular.io/docs/ts/latest/](https://angular.io/docs/ts/latest/)

## Pre-requisites
* Install [node](https://nodejs.org/en/download/) & [npm](https://www.npmjs.com/)
* Install [angular-cli](https://github.com/angular/angular-cli/): `npm install -g @angular/cli`
* Install [gulp](https://github.com/gulpjs/gulp): `npm install -g gulp`

## Project set-up
Follow these steps (taken directly from the [angular-cli](https://github.com/angular/angular-cli/) page) to create a blank project, and run it locally. Keep it running for the rest of this dojo:

* `ng new angular-dojo-frontend`
* `cd angular-dojo-frontend`
* `ng serve`

### Styling
For this dojo, we'll use [Semantic UI](https://semantic-ui.com/) as our CSS framework. First we need to install Semantic UI, and build the CSS and JS:

* `npm install semantic-ui jquery --save` (Note: due to a quirk in the semantic setup, you'll need to run this in cmd rather than git bash. Install using all the defaults)
* `cd semantic`
* `gulp build`

Then, reference the compiled semantic CSS and JS, along with jquery, by editing `angular-cli.json`:

```
"apps": [{
  ... 
  "styles": [
      "styles.css",
      "semantic/dist/semantic.min.css"
  ],
  "scripts": [
      "../node_modules/jquery/dist/jquery.min.js",
      "semantic/dist/semantic.min.js"
  ],
  ...
}]
```

See if it's working, by editing `app.component.html`:

* Wrap the contents of the component in a `div`, with class `ui container segment`
* Edit the `h1`, so it has class `ui header`
* Add a `p` below the `h1`, with a bit of content

## Binding
Let's play around with some Angular binding, by editing `app.component.html`:

* Add `<input [(ngModel)]="title">` to the component

Note that this is a **two way binding**. Notice how the value displayed in the `<h1>` (which is using a **one way binding**) automatically changes in real time.

## Components
Create a new navigation component using angular-cli:

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
* Bask in your own glory. For bonus points - look into Angular's **Child Routes**

## Services
Let's create a service, so that we can put some data on the screen:

* `ng g service data-access`
* Edit the new service, and give it a method that returns some values. Something like:
```
getValues(): string[] {
  return ["1", "2"];
}
```
* **Inject** the new service into `about.component.ts`. **Don't** instantiate it!
* **Provide** the new service in `about.component.ts`. **Hint:** `providers: [DataAccessService]`
* Call the new `getValues()` method when the `about` component initiates (the lifecycle hook should already be there, thanks to angular-cli). Assign the return value to a local variable (say `values`)
* Edit `about.component.html`, to actually display the values. Here we can use some Angular syntax, like:
```
<div *ngFor="let value of values">
  {{value}}
</div>
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
* In `about.component.html`, display something when loading. (Hint: `*ngIf="isLoading"`)

## Interfacing with an API
This repository contains a pre-built dotnet core webapi project. Open the solution within `./api/src/` in Visual Studio, and run the webapi project. Keep it running in the background.

### Swagger
With the webapi project running, you should be able to browse to the Swagger endpoints, thanks to [NSwag](https://github.com/NSwag/NSwag):

* Swagger UI: http://localhost:4201/swagger
* Swagger JSON: http://localhost:4201/swagger/v1/swagger.json

### NSwag client
Let's use NSwag to generate a TypeScript client for us:

* Install NSwag 8.0.0*: `npm install nswag@8.0.0 -g`
* Generate the TS client: `nswag swagger2tsclient /input:http://localhost:4201/swagger/v1/swagger.json /output:./src/api/apiclient.ts /template:angular2`
* Edit `about.component.ts`:
  * Add import: `import * as apiClient from '../../api/apiclient';`
  * Add provider: `providers: [apiClient.ValuesClient]`
  * Change injected service: `valuesClient: apiClient.ValuesClient`
  * Change method call: `this.valuesClient.getAll().subscribe(result => ...`

Check that the data shows up on the screen.

*For some reason the latest version of NSwag wasn't working properly at the time of writing, but 8.0.0 was. Feel free to try the latest version if you want. Note that the template name is simply `angular`, not `angular2` when using the latest version.

### Posting and waiting
TODO - post to API. Disable `<form>` whilst waiting for response

## Pipes
Let's check out Angular's pipes:

* Edit `home.component.ts` - add a local variable `exampleDate: Date = new Date();`
* Edit `home.component.html`- display `exampleDate`, using `{{exampleDate}}`

Notice how the full Date object is displayed. Let's change that with a pipe:

* Change the binding to `{{ exampleDate | date }}`

Ooh shiny. You can of course create your own pipes, and you can parameterise them too.

## Attribute directives
Let's check out Angular's attribute directives. Create a new directive using angular-cli:

* `ng g directive highlight`

Now edit  `highlight.directive.ts`:

* Inject `private el: ElementRef` into the constructor
* Make the directive implement `OnInit`, and add the `ngOnInit` method. Edit it to something like:
```
ngOnInit() {
  this.el.nativeElement.style.backgroundColor = 'yellow';
}
```

Now let's actually use our directive, by adding it as an attribute to any HTML element:
```
<div appHighlight>
```

We can also add `@Input` properties onto directives, for further customisability. Edit `highlight.directive.ts`:

* Add an input property: `@Input() highlightColor: string;`
* Use this new property, instead of hard-coding `"yellow"`, in `ngOnInit()`

Now we can use our directive along with its input property:
```
<div appHighlight highlightColor="yellow">
```

**Bonus points:** make the syntax more compact, so you can just write `<div appHighlight="yellow">`. You can do this by naming the `@Input` property appropriately, and using an `@Input` **alias** inside the directive, for readability.

## Unit tests
TODO

## Useful tools

* [Augury](https://chrome.google.com/webstore/detail/augury/elgalmkoelokbchhkhacckoklkejnhcd?hl=en)