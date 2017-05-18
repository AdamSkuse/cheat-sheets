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

*app/styles/modules* <-- css modules (e.g. *_footer.css*) go in here

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

## Browsersync

This will allow us to view changes in browser without having to hit reload.

**1.** `npm install browser-sync --save-dev`

**2.** in *gulpfile.js* add `browserSync = require('browser-sync’).create();`

and edit watch task to become:
```
gulp.task('watch', function() {

  browserSync.init({
    server: {
      baseDir: "app"
    }
  });

  watch('./app/index.html', function() {
    browserSync.reload();
  });

  watch('./app/assets/styles/**/*.css', function() {
    gulp.start(‘cssInject’);
  });
});
```
and add new task at bottom of *gulpfile.js*:

```
gulp.task('cssInject', ['styles'], function() {
  return gulp.src('app/temp/styles/**/*.css')
    .pipe(browserSync.stream());
});
```

**3.** Optional refactoring of gulpfile by putting tasks into *gulp/tasks* as separate files e.g. *watch.js*, *styles.js*
then in *gulpfile.js* add:

```
require('./gulp/tasks/styles');
require('./gulp/tasks/watch');
```

**4.** Add error handling for styles task, so css syntax errors won't cause browsersync/watch task to halt.

Amend *gulp/tasks/styles.js* as follows:

```
gulp.task('styles', function() {
  return gulp.src('./app/assets/styles/styles.css')
    .pipe(postcss([cssImport, cssvars, nested, autoprefixer]))
    .on('error', function(errorInfo) {
	console.log(errorInfo.toString());
      this.emit('end');
    })
    .pipe(gulp.dest('./app/temp/styles'));
});
```


## Post-css mixins

For writing succinct media queries for mobile responsiveness.
In css, start with styles for baseline smallest screen size, then add mixin mediaqueries for progressively large screens.

**1.**
```
npm install postcss-mixins --save-dev
```

add following to *gulp/tasks/styles.js*:
`mixins = require('postcss-mixins');`

and update pipe in same file:
`.pipe(postcss([cssImport, mixins, cssvars, nested, autoprefixer]))`

**2.** create *app/assets/styles/base/_mixins.css* and add following:
```
@define-mixin atSmall {
  @media (min-width: 530px) {
    @mixin-content;
  }
}

@define-mixin atMedium {
  @media (min-width: 800px) {
    @mixin-content;
  }
}

@define-mixin atLarge {
  @media (min-width: 1200px) {
    @mixin-content;
  }
}
```

be sure to import this file in *styles.css*

**3.** Then use `@mixin [mixinName]` in stylesheets. Nest in selectors as follows:

```
.large-hero {

	&__subtitle {
		font:size: 1.1rem;

		@mixin atSmall {
			font-size: 4.8rem;
			}
	}

}
```
