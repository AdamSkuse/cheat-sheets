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

## Responsive images
For dealing with two issues:
1. Art direction (e.g. using differently-cropped images depending on screen size.
2. File size / resolution (so only images with resolution/size appropriate to the user's screen are downloaded)

**1. "Art direction"**
Use the `<picture>` tag with media attribute:
```
<picture>
	<source srcset=“images/dog-crop-large.jpg” media=“(min-width: 1200px)”>
	<source srcset=“images/dog-crop-medium.jpg” media=“(min-width: 760px)”>
	<img src=“images/dog-crop-small.jpg” alt=“Puppy in the sand.”>
</picture>
```

**2. Image resolution / file size**
Use `srcset`. The number after each image is the width of the image itself. The browser/device uses this to determine which is the most appropriate to download, and doesn’t download the alternatives.
```
<img srcset=“images/dog-res-small.jpg 570w, images/dog-res-medium.jpg 1200w, images/dog-res-large.jpg 1920w” alt=“Puppy in the sand”>
```

## Object-oriented Javascript and Webpack

We can create "classes" as follows:
```
function Person(fullName, favColor) {
this.name = fullName;
this.favoriteColor = favColor;
this.greet = function() {
	console.log(“Hi, my name is “ + this.name + “ and my fav color is “ + this.favoriteColor “ .”;
	}
}

var john = new Person(“John Doe”, “blue”)
john.greet();
```

As JS doesn’t have a native ‘require’ function, we can use **Webpack**. This will allow us to store class constructors etc in separate .js files to our main code i.e. make it modular.

**1.** Create:
*app/assets/scripts/App.js*
*app/assets/scripts/modules/*

Modules (e.g. *Person.js*) will go in the *modules/* folder. We then require them at the top of *App.js* e.g.
`var Person = require(‘./modules/Person’);`

Webpack will compile all the js modules into a single file that the browser can then read.

**2.** if not already installed globally, do so now with `npm install webpack -g`

**3.** In the project root folder (i.e. alongside *package.json*) create *webpack.config.js*, then add the following to it:
```
module.exports = {
	entry: “./app/assets/scripts/App.js”,
	output: {
		path: “./app/temp/scripts”,
		filename: “App.js”
		}
}
```

**4.** Update path to script file in *index.html* to point to *assets/temp/scripts/App.js*

N.B. each js module (e.g. Person.js) will need an export line at the end, so that Webpack knows what to do with it. e.g.
`module.exports = Person;`

**5.** Webpack can also require third-party modules such as jquery. If using jquery in project, first install it with `npm install jquery --save`. Then, in each module that uses jquery, at the top put `var $ = require('jquery');

#### Adding Webpack to Gulp workflow
This will set browsersync to automatically update if a .js file is edited:

**1.** `npm install webpack --save-dev`

**2.** create *gulp/tasks/scripts.js*

**3.** in *gulpfile.js* add `require(‘./gulp/tasks/scripts’);`

**4.** add following to *gulp/tasks/scripts.js*

```
var gulp = require(‘gulp’),
webpack = require(‘webpack’);

gulp.task(‘scripts’, function(callback) {
	webpack(require(‘../../webpack.config.js’), function(err, stats) {
		if (err) {
			console.log(err.toString());
		}
		console.log(stats.toString());
		callback();

	});
});
```

**5.** In *watch.js*, after 'css' and 'watch' tasks, add:

```
watch(‘./app/assets/scripts/**/*.js), function() {
	gulp.start(‘scriptsRefresh’);
})
```

and after 'cssInject' task add:

```
gulp.task(‘scriptsRefresh’, [‘scripts’], function() {
	browserSync.reload();
)};
```

Now, webpack will compile a js file for the browser to use from our main file and modules into App.js in the temp folder; and browserSync will automatically update the browser when any js files are edited. If there are any errors it will throw them up without causing browsersync to end.
