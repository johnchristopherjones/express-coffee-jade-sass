This Gulpfile is sort of like a Makefile.  It defines various
automation tasks.

First, load all packages required to get started.  Sourcemaps
are used for most tasks, so we also load those.

    gulp       = require 'gulp'
    sourcemaps = require 'gulp-sourcemaps'

Several of the Gulp modules can blow up in ways that break
the watch task.  Since we don't want the 'watch' task to stop
watching just because something went wrong, we create a function
to handle the error.

    handle = (stream)->
        gutil = require 'gulp-util'
        stream.on 'error', ->
            gutil.log.apply this, arguments
            gutil.beep()
            stream.end()

We'll compile client-side coffeescript and include sourcemaps.
Sourcemaps let us map our deployment-mangled JS files back to
our beautiful coffeescript files.  Chrome knows how to use
sourcemaps.

    gulp.task 'coffee', ->
        coffee = require 'gulp-coffee'
        concat = require 'gulp-concat'
        uglify = require 'gulp-uglify'
        gulp.src 'client/scripts/*.coffee.md'
            .pipe sourcemaps.init()
            .pipe handle coffee()
            .pipe uglify()
            .pipe concat 'script.js'
            .pipe sourcemaps.write()
            .pipe gulp.dest 'public/scripts'

We also want to lint our CoffeeScript whenever possible. This
helps us detect potential problems sooner than later.  Linting
is especially helpful if you're new to CoffeeScript.

    gulp.task 'lint', ->
        coffeelint = require 'gulp-coffeelint'
        gulp.src 'client/scripts/*.coffee.md'
            .pipe handle coffeelint()
            .pipe coffeelint.reporter()

Generating CSS from SCSS (SASS) is similar to compiling
CoffeeScript.  We can also include sourcemaps to map the
deployed files back to the original source files.  Again,
Chrome knows how use sourcemaps.

We can tell auto-prefixer the browsers that we want to target.

    gulp.task 'style', ->
        sass   = require 'gulp-sass'
        cssmin = require 'gulp-minify-css'
        prefix = require 'gulp-autoprefixer'
        gulp.src 'client/styles/*'
            .pipe sourcemaps.init()
            .pipe handle sass()
            .pipe prefix browsers: ['> 1%']
            .pipe cssmin keepSpecialComments: 0
            .pipe sourcemaps.write()
            .pipe gulp.dest 'public/styles'

We don't need to compile Jade for Express because we use some
middleware that handles it.  However, if we want to deploy
static HTML files that are built from Jade, we need this.
Thankfully, we can use sourcemaps here, as well.

    gulp.task 'html', ->
        jade       = require 'gulp-jade'
        minifyHTML = require 'gulp-minify-html'
        gulp.src 'client/*.jade'
            .pipe sourcemaps.init()
            .pipe handle jade()
            .pipe minifyHTML()
            .pipe sourcemaps.write()
            .pipe gulp.dest 'public'

SVG should be minified, if any.

    gulp.task 'svg', ->
        svgmin = require 'gulp-svgmin'
        gulp.src 'client/images/*.svg'
            .pipe svgmin()
            .pipe gulp.dest 'public/images'

We can copy any fonts we've chosen to include.

    gulp.task 'fonts', ->
        gulp.src 'client/fonts/*'
            .pipe gulp.dest 'public/fonts'


Generate documentaiton using docco.  Documentation is generated
automatically from literate coffeescript.

    gulp.task 'docs', ->
        docco = require 'gulp-docco'
        options =
            # layout: 'classic'
            layout: 'linear'
            # layout: 'parallel'
        # Compile client documentation
        gulp.src 'client/scripts/*.coffee.md'
            .pipe handle docco(options)
            .pipe gulp.dest 'docs/client'
        # Compile server documentatin
        gulp.src '*.coffee.md'
            .pipe handle docco(options)
            .pipe gulp.dest 'docs/server'
        gulp.src 'routes/*.coffee.md'
            .pipe handle docco(options)
            .pipe gulp.dest 'docs/server/routes'

Express starts/restarts the server.  To enable livereload, uncomment
the livereload lines.

    gulp.task 'server:restart', ->
        server = require 'gulp-develop-server'
        # livereload = require 'gulp-livereload'
        gulp.src 'app.coffee.md'
            .pipe server
                path: './app.coffee.md'
                execArgv: ['--harmony']
            # .pipe livereload()

Gulp watch will watch our source files and recompile them as
they are changed.  It will also restart our server for us if our
server files change.

    gulp.task 'watch', ->

Our static client files are coffee, jade, and sass, so we need to
recompile and deploy them for static hosting.  Note that the
dynamic files are served from /views/ and do not need to be
recompiled.

        gulp.watch 'client/scripts/*.coffee.md',
            ['coffee', 'lint', 'docs']
        gulp.watch 'client/*.jade', ['html']

Our styles can be either SASS or SCSS.  SASS is better for consistency
with the style of jade and coffeescript, but SCSS is fine.

        gulp.watch 'client/styles/*', ['style']
        gulp.watch 'client/images/*.svg', ['svg']
        gulp.watch 'client/fonts/*', ['fonts']

If our server files change, we should restart the server.

        gulp.watch '*.coffee.md',
            ['lint', 'docs', 'server:restart']
        gulp.watch 'routes/*.coffee.md',
            ['lint', 'docs', 'server:restart']

The default task just compiles static files.

    gulp.task 'default',
        ['html', 'style', 'coffee', 'svg', 'fonts', 'server:restart', 'watch']
