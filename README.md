# ES6 Basics

## Agenda

1. Syntax
1. Organization
1. Compatibility
1. Demo

## Syntax

### `let`

`let` is the same as `var`, except within a `for` loop:

```javascript
function oldShit() {
    // i is visible out here
    for( var i = 0; i < 5; i++ ) {
        // i is visible to the whole function
    };
    // i is visible out here
};

function newShit() {
    // not visible out here
    for( let i = 0; i < 5; i++ ) {
        // i is only visible in here (and in the for() parentheses)
    };
    // nope, i is not visible here either.
};
```

### `class`

```javascript
class Shape {
    constructor() {
        // this runs when you call new Shape()
        this.shapeWidth = 0;

        this.draw = function() {
            //available only after you call new Shape()
        };
    }
    this.width(x) {
        //available on Shape's prototype without
        //calling new Shape()
        if(x !== undefined){
            this.shapeWidth = parseInt(x,10);
        }
        return this.shapeWidth;
    }
}
export default Shape;
```

### `extends`

```javascript
import Shape from './Shape.es6';

class Square extends Shape {
    constructor() {
        super() // this runs Shape's constructor
        // do things unique for Square here.
    }
}
export default Square;
```

## Organization

Let's put this together.

### Application bootstrap

```
import $ from './lib/jquery-1.11.2.min';
import ModuleLoader from './modules/ModuleLoader.es6';
import Utils from './modules/Utils.es6';
import Foo from './modules/Foo.es6';
import Bar from './modules/Bar.es6';

let app = new ModuleLoader($, Utils,
                        [Foo, Bar], $('[data-module]'));
app.init();
```

### ModuleLoader

```javascript
class ModuleLoader {
    constructor($, Utils, imports, views) {
        this._imports = imports;
        this._modulesLoaded = 0; //expose for testing

        let getModulesOnPage = function() {
            let _modules = [],
                _module;
            for (let i = 0; i < views.length; i++){
                _module = $(views[i]).data('module');
                _modules.push(_module);
            }
            return _modules;
        };

        this.instantiateModules = function(modules) {
            let _modulesOnPage = getModulesOnPage(),
                _module,
                _name,
                _loaded = [];

            for (let i = 0; i < modules.length; i++){
                _name = Object.create(modules[i]).prototype.name();
                if(_modulesOnPage.indexOf(_name) > -1
                    && _loaded.indexOf(_name) === -1){
                    _module = new modules[i]($, Utils);
                    _module.init();
                    _loaded.push(_name);
                    this._modulesLoaded++; //this is for testing
                }
            }
        };
    }

    init() {
        this.instantiateModules(this._imports);
    }
}
export default ModuleLoader;
```

### Controller

```javascript
class Foo {
    constructor($, Utils) {
        let utils = new Utils();
        //constants, methods, module setup, etc

        this.start = function() {
            //event listeners or whatever
        };
    }

    name() {
        return "Foo";
    }

    init() {
        this.start();
    }
}
export default Foo;
```

### View

```javascript
<section class="foo" data-module="Foo">;
    <!-- module markup here -->;
</section>;
```

## Compatibility

But, I have to support IE9 :(

### No problem!

We can transpile to ES5.

### WTF is transpile?

Translate + Compile = Transpile. We can use a task runner, like `grunt` or `gulp` to convert this to IE9 speak.

```javascript
var gulp = require('gulp'),
    browserify = require('gulp-browserify2'),
    babelify = require('babelify'),
    jshint = require('gulp-jshint'),
    buffer = require('vinyl-buffer'),
    sourcemaps = require('gulp-sourcemaps'),
    rename = require('gulp-rename');

gulp.task('lint', function () {
  gulp.src('./js/*.es6.js')
      .pipe(jshint({
          esnext: true,
          expr: true,
          jquery: true
          }))
      .pipe(jshint.reporter('default'))
      .pipe(jshint.reporter('fail'));
});

gulp.task('transpile', function() {
  gulp.src('./js/app.es6.js')
  .pipe(browserify({
      fileName: 'app.js',
      transform: {
          tr: babelify,
          options: {
              loose: ['es6.modules']
          }
      },
      options: {
          debug: true
      }
  }))
  .pipe(buffer())
  .pipe(sourcemaps.init({ loadMaps: true }))
  .pipe(sourcemaps.write('./'))
  .pipe(gulp.dest('./public/Includes/scripts'));
});

gulp.task('watch', function() {
    gulp.watch(['./js/**/*.es6.js'], ['lint','transpile']);
});

gulp.task('default', ['lint', 'transpile', 'watch']);
```

## Demo

https://github.com/mjhea0/es6-node-mongo-boilerplate
