# Express Coffee/Sass/Jade Template

This is a Node Express template stack.

If you've ever wanted to go "full stack" with CoffeeScript, Jade, and Sass (or SCSS), this is for you.  The server, the client scripts, and even the Gulpfile are written in Literate CoffeeScript.

You can mix Jade middleware with Jade static resources.

Documentation is automatically generated with `docco`.

# Installation

1. Clone this repository.

2. Install the node requirements:

       npm install

3. Generate static files and start the server:

       gulp

   or, in production if your static files have already been generated:

       npm start

# Generating documentation

While `gulp` is watching, all of your coffeescript files will be compiled to docs in the `docs` folder as they are saved.

If you want to generate documentation all at once, just run

    gulp docs

# How things are organized

The source files for your static resources are in `/client`.  Everything that goes into that folder will be preprocessed.

- Static HTML files are generated from `*.jade` in '/client'
- Static CSS files are generated from `*.scss` and `*.sass` in `/client/styles`
- Static JS files are generated from `*.coffee.md` in `/client/scripts`
- Static fonts and images are copied from `/client/fonts` and `/client/images`

The preprocessed files are sourcemapped, minified, uglified, and (where appropriate) concatenated.  Then, each file type is placed in parallel folders in `/public`.  Scripts and stylesheets are joined into singular files.  If you outgrow this, modify the Gulpfile.

Each of your routes should be a separate file in `/routes`.  Your views are jade files in `/views` that you can render from your routes.  You'll need to modify `app.coffee.md` to bind your new route when you add one.  You bind a new route by adding lines to `app.coffee.md` similar to the following.

    users = require './routes/users'
    app.use '/users', users
