# Angular dojo
It's time to do some Angular!

## Pre-requisites
* Install [npm](https://www.npmjs.com/)
* Install [angular-cli](https://github.com/angular/angular-cli/) (`npm install -g @angular/cli`)
* Install [gulp](https://github.com/gulpjs/gulp) (`npm install -g gulp`)

## Project set-up
Follow the steps on the [angular-cli](https://github.com/angular/angular-cli/) page to create a blank project, and run it locally. Keep it running for the rest of this dojo.

### Styling
For this dojo, we'll use [Semantic UI](https://semantic-ui.com/) as our CSS framework. First we need to install Semantic UI, and build the CSS and JS:

* `npm install semantic-ui --save-dev` (Note: due to a quirk in the semantic setup, you'll need to run this in cmd rather than git bash. Install using all the defaults.)
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

* Wrap the contents of the component in a `div`, with class `ui raised container segment`
* Edit the `h1`, so it has class `ui header`
* Add a `p` below the `h1`, with a bit of content

## Create a component
Create a navigation component using angular-cli:

* `ng g component navigation`

Notice how it creates the scaffolding for our new component, and also edits our app definition in `app.module.ts`. Now let's add it into our app:

* Edit `/navigation/navigation.component.html`. Add a semantic menu, something like:
```
<div class="ui top menu">
  <a class="item">Home</a>
  <a class="item">Somewhere else</a>
</div>
```
* Add `<app-navigation></app-navigation>` to the top of `app.component.html`.

## Routing
TODO

## Create a directive
TODO

## Create a pipe
TODO

## Create a service
TODO

## Unit tests
TODO

## Interfacing with an API
This repository contains a pre-built dotnet core webapi project. Open the solution within `./api/src/` in Visual Studio, and run the webapi project. Keep it running in the background.

### Swagger
With the webapi project running, tou should be able to browse to the Swagger endpoints, thanks to [NSwag](https://github.com/NSwag/NSwag):

* Swagger UI: http://localhost:4201/swagger
* Swagger JSON: http://localhost:4201/swagger/v1/swagger.json
