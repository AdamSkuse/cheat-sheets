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

**6.** Set up **post-css** with **autoprefixer** (so we don't have to worry about webkit prefixes to ensure browser css compatibility); **simple-vars** (so we can use variables in our css); **nested** (so we can nest selectors in our css) and **import** (so we can write modular css across different files).

```
npm install gulp-postcss --save-dev
npm install autoprefixer --save-dev
npm install postcss-simple-vars --save-dev
npm install postcss-nested --save-dev
npm install postcss-import --save-dev
```

add to *gulpfile.js*:
```
postcss = require('gulp-postcss’),
autoprefixer = require(‘autoprefixer’),
cssvars = require('postcss-simple-vars’),
nested = require('postcss-nested’),
cssImport = require('postcss-import');


gulp.task('styles', function() {
  return gulp.src('app/assets/styles/styles.css')
    .pipe(postcss([cssImport, cssvars, nested, autoprefixer]))
    .pipe(gulp.dest('app/temp/styles'));
});
```

**7.** add following to *app/index.html*:

```
<link rel="stylesheet" href="temp/styles/styles.css">
```

**8.** create:
*app/styles/base*  <-- our *_global.css* goes in here
*app/styles/modules** <-- css modules (e.g. *_footer.css*) go in here

**9.** import css modules in *_global.css* e.g.
```
@import “base/_global.css”;
@import “modules/_footer.css”;
```

**not on nested css
to avoid nested css compiling as nested selectors when using BEM, use ampersands as follows:
```
.large-hero {
	position: relative;

	&__text-content {
		position: absolute;
	}
}

The above will be compiled by post-css into the final styles.css as:

.large-hero {
	position: relative;
}

.large-hero__text-content {
	position: absolute;
}
```
