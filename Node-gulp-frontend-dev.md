# Node Gulp Frontend Dev cheat sheet
### Based on the excellent Git A Web Developer Job course on Udemy by Brad Schiff

N.B. this cheat sheet skips the use of modernizr and measures for dealing with SVG files. It also skips info on Git / Github, as I was already familiar with these at the time of doing the course.

Also, be careful if copying-and-pasting code snippets from this doc, as many of the quote marks are encoded as 'curly', so will throw errors if not fixed.

You can read my thoughts on the course itself at 
http://adamskuse.com/blog/coding/2017/05/22/thoughts-on-git-a-web-developer-job-udemy-course-by-brad-schiff/

5/24/17
I will be adding extra sections/material/resources to this sheet that are not based on Brad's course as I develop my own workflow.


## Document contents

[Basics](#basics)

[Initializing a new project](#initializing-a-new-project)

[Browsersync](#browsersync)

[Post-css mixins](#post-css-mixins)

[Responsive images](#responsive-images)

[Object-oriented Javascript and Webpack](#object-oriented-javascript-and-webpack)

[Babel](#babel)

[Lazy loading images](#lazy-loading-images)

[Prepping files to go live](#prepping-files-to-go-live)

[Useful resources](#useful-resources)

[Example project folder structure](#example-project-folder-structure)

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

**note on nested css:**
to avoid nested css compiling as nested selectors when using BEM, use ampersands as follows:
```
.large-hero {
	position: relative;

	&__text-content {
		position: absolute;
	}
}
```
The above will be compiled by post-css into the final styles.css as:
```
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
and

`npm install gulp-watch --save-dev`

**2.** in *gulpfile.js* add 
```
watch = require('gulp-watch')
browserSync = require('browser-sync').create();
```

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
*N.B. The above is the ECMA5 approach to creating 'classes'. Later, after installing Babel, we can use the ECMA6 class constructor (see section on Babel).*

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

## Babel
Allows us to write ES6 code, which Babel then converts to ES5 code to ensure browser compatibility.

**1.** `npm install babel-core babel-loader babel-preset-es2015 --save-dev`

**2.** In *webpack.config.js* add a module so the whole thing reads as follows:
```
module.exports = {
	entry: “./app/assets/scripts/App.js”,
	output: {
		path: “./app/temp/scripts”,
		filename: “App.js”
		},
		modules: {
			loaders: [
				{
					loader: ‘babel’,
					query: {
						presets: [‘es2015’]
					},
					test: /\.js$/,
					exclude: /node_modules/
					}
				]
			}
}
```
this means Webpack will run the code thru Babel before creating the dist version of App.js.

#### ECMA6 refactor - classes
We can now update our classes to use ECMA6 as follows:

```
class Person {
	constructor(fullName, favColor) {
		this.name = fullName;
		this.favoriteColor = favColor;
	}

	greet() {
	console.log(“Hi, my name is “ + this.name + “ and my fav color 	is “ + this.favoriteColor “ .”;
	}
}

module.exports = Person;
```

#### ECMA6 refactor - imports / exports
Additionally, as ECMA6 supports importing from external .js files, in *App.js* we can replace

`var Person = require(‘./modules/Person’);`

with

`import Person from ‘./modules/Person’;`

and in *Person.js* replace

`module.exports = Person;`

with

`export default Person;`

## Lazy loading images

**1.** `npm install lazysizes --save`

**2.** create *Vendor.js* in same dir as *App.js* and add line `import 'lazysizes';`

**3.** In *webpack.config.js* edit 'entry' and 'export' properties to following:
```
module.exports = {
	entry: {
			App: “./app/assets/scripts/App.js”,
			Vendor: “./app/assets/scripts/Vendor.js”
		},
	output: {
		path: “./app/temp/scripts”,
		filename: “[name].js”
		},
		modules: {
			loaders: [
				{
					loader: ‘babel’,
					query: {
						presets: [‘es2015’]
					},
					test: /\.js$/,
					exclude: /node_modules/
					}
				]
			}
}
```

**4.** in head of *index.html* add `<script src=“/temp/scripts/Vendor.js”></script>`

**5.** To set an image to lazyload, add the class 'lazy-load' and change 'src'/'srcset' to 'data-src'/'data-srcset'. e.g.:

`<img class=“lazyload” data-srcset=“image1.jpg 160w, image2.jpg 320w”>`

**6.** For background images, add 'lazyload' class to the html element. Then in the css assign the background image in a 'lazyloaded' class. This class will be added to the html element when the window scrolls close to the element. e.g.

```
.testimonials {
	&.lazyloaded {
		background: url(/assets/images/background.jpg) top 			center no-repeat;
		background-size: cover;
	}
}
```

## Prepping files to go live

**1.** create *dist* folder

**2.**
Install imagemin for image compression
`npm install gulp-imagemin --save-dev`

Install del so we can delete old files when creating a new build
`npm install del --save-dev`

**3.** create *gulp/tasks/build.js*

#### Process images

**1.** Add following to *gulp/tasks/build.js*

```
var gulp = require('gulp'),
imagemin = require('gulp-imagemin');

gulp.task('optimizeImages', function() {
  return gulp.src("./app/assets/images/**/*")
  .pipe(imagemin({
    progressive: true,
    interlaced: true,
    multipass: true
  }))

  .pipe(gulp.dest("./dist/assets/images"));
});

gulp.task('build', ['optimizeImages']);
```

**2.** In *gulpfile.js* add `require(‘./gulp/tasks/build’);

**3.** Add 'deleteDistFolder' task to *build.js* to clear out dist folder before each new build

```
var gulp = require('gulp'),
imagemin = require('gulp-imagemin'),
del = require('del'); //so we can delete the dist folder at start of each build

gulp.task('deleteDistFolder', function() {
  return del("./dist");
});

gulp.task('optimizeImages', ['deleteDistFolder'], function() {
  return gulp.src("./app/assets/images/**/*")
  .pipe(imagemin({
    progressive: true,
    interlaced: true,
    multipass: true
  }))

  .pipe(gulp.dest("./dist/assets/images"));
});

gulp.task('build', ['deleteDistFolder', 'optimizeImages']);

```
N.B. 'deleteDistFolder' has been added as a dependency in the last line.

#### Process CSS and Javascript with Usemin
We want to
-copy to dist folder
-compress
-revision (make unique filename that forces browsers that have cached our page to d/load new versions)

**1.** `npm install gulp-usemin —save-dev`

**2.** add to *build.js* so it looks like following:
```
var gulp = require('gulp'),
imagemin = require('gulp-imagemin'),
del = require('del'), //so we can delete the dist folder at start of each build
usemin = require('gulp-usemin');

gulp.task('deleteDistFolder', function() {
  return del("./dist");
});

gulp.task('optimizeImages', ['deleteDistFolder'], function() {
  return gulp.src("./app/assets/images/**/*")
  .pipe(imagemin({
    progressive: true,
    interlaced: true,
    multipass: true
  }))

  .pipe(gulp.dest("./dist/assets/images"));
});

gulp.task('usemin', ['deleteDistFolder'], function() {
  return gulp.src("./app/index.html")
  .pipe(usemin())
  .pipe(gulp.dest("./dist"));
});

gulp.task('build', ['deleteDistFolder', 'optimizeImages', 'usemin']);
```

**3.** So that the dist version of *index.html* points to the correct, dynamically-named versions of the CSS and JS files, wrap the references in comments as follows:
```
<!-- build:css assets/styles/styles.css -->
<link rel="stylesheet" href="temp/styles/styles.css">
<!-- endbuild -->
```
```
<!-- build:js assets/scripts/App.js -->
<script src="assets/scripts/App.js"></script>
<!-- endbuild -->
```

**4.** run `gulp build`
this will create index.html in dist folder with usemin comments removed, and create script and css files in appropriate dist subfolders.

#### Compression and versioning

**1.** `npm install gulp-rev gulp-cssnano gulp-uglify --save-dev`
(gulp-rev revisions files; gulp-cssnano compresses css; gulp-uglify compresses javascript)

**2.** at top of *build.js* add:
```
rev = require('gulp-rev'),
cssnano = require('gulp-cssnano'),
uglify = require('gulp-uglify');
```

**3.** Update 'usemin' task:
```
gulp.task('usemin', ['deleteDistFolder'], function() {
  return gulp.src("./app/index.html")
  .pipe(usemin({
    css: [function() {return rev()}, function() {return cssnano()}],
    js: [function() {return rev()}, function() {return uglify()}]
  }))
  .pipe(gulp.dest("./dist"));
});
```
NB If you're using ECMA6 syntax (e.g. arrow functions) in your JS and are not using Babel, uglify will throw an error.

**4.** Add a previewDist task to *build.js* that will let us preview our dist files using Browsersync:

```
var gulp = require('gulp'),
imagemin = require('gulp-imagemin'),
del = require('del'), //so we can delete the dist folder at start of each build
usemin = require('gulp-usemin'),
rev = require('gulp-rev'),
cssnano = require('gulp-cssnano'),
uglify = require('gulp-uglify'),
browserSync = require('browser-sync').create();

gulp.task('previewDist', function() {
  browserSync.init({
    notify: false,
    server: {
      baseDir: "dist"
    }
  });
});
gulp.task('deleteDistFolder', function() {
  return del("./dist");
});

gulp.task('optimizeImages', ['deleteDistFolder'], function() {
  return gulp.src("./app/assets/images/**/*")
  .pipe(imagemin({
    progressive: true,
    interlaced: true,
    multipass: true
  }))

  .pipe(gulp.dest("./dist/assets/images"));
});

gulp.task('usemin', ['deleteDistFolder', 'styles'], function() {
  return gulp.src("./app/index.html")
  .pipe(usemin({
    css: [function() {return rev()}, function() {return cssnano()}],
    js: [function() {return rev()}]
  }))
  .pipe(gulp.dest("./dist"));
});

gulp.task('build', ['deleteDistFolder', 'optimizeImages', 'usemin']);
```
note that  styles was added as a dependency of usemin. if we had a scripts task, as in the original tutorial, this would also be added as a dependency here.

#### Copying other files to our dist folder with build
**1.** Add the following to *build.js*:
```
gulp.task('copyGeneralFiles', ['deleteDistFolder'], function(){
  var pathsToCopy = [
    './app/**/*',
    '!./app/index.html',
    '!./app/assets/images/**',
    '!./app/assets/styles/**',
    '!./app/assets/scripts/**',
    '!./app/temp',
    '!./app/temp/**'
  ]

  return gulp.src(pathsToCopy)
  .pipe(gulp.dest("./dist"));
});
```
**2.** Update build task at end of same file:
```
gulp.task('build', ['deleteDistFolder', 'copyGeneralFiles', 'optimizeImages', 'usemin']);
```
## Useful resources

https://www.sitepoint.com/anatomy-of-a-modern-javascript-application

https://www.sitepoint.com/introduction-gulp-js/


## Example project folder structure
```
project-dir
	package.json
	gulpfile.js

	|-app
		index.html

		|- assets
			|-fonts
			|-images
			|-styles
				styles.css
				|-base
					_global.css
				|-modules
					_header.css
					_footer.css
			|-scripts
				App.js
				|-modules

		|-temp
			|-styles
	|-dist
		index.html
		|-assets
			|-fonts
			|-images
			|-scripts
			|-styles
	|-misc
	|-node_modules
```
