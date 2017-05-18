# Node Gulp Frontend Dev cheat sheet
### Based on the excellent Git A Web Developer Job course on Udemy by Brad Schiff

## Basics

#### Node
A runtime env for Javascript that allows javascript to be used serverside (backend) or on a dev computer to automate tasks.

#### NPM
Node Package Manager. A repository of Node code, as well as third-party libraries such as Bootstrap and jQuery. Allows for easy installation and updating from the command line.

#### Gulp
A JS-based build system for automating dev tasks using gulp plugins.

## Initializing a new project

**1.** Run `npm init` in project folder. This sets up NPM for the project, and creates *package.json*, which will store config info on the packages used in your project.

**2.** Run `npm install normalize.css --save`. This installs normalize.css, a css reset.

**3.** If jQuery required, `npm install jquery --save-dev`

**4.** Install gulp. `npm install gulp --save-dev`

**5.** Create *gulpfile.js* in project root. Add placeholder tasks to the file:

```
var gulp = require(‘gulp’);

	gulp.task('default', function() {
  		console.log("default task");
	});

	gulp.task(‘html’, function() {
		console.log(“html task”);
	});

	gulp.task(‘styles’, function() {
		console.log(“styles task”);
	});
```
