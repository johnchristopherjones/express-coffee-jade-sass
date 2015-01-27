Welcome to the application!  This is literate coffeescript based upon
[Zaiste's article from 2012][ZAISTE 2012].

[ZAISTE 2012]: http://zaiste.net/2012/11/\
restful_api_with_express_mongoose_on_coffeescript/

Load requirements, inluding Express and BodyParser.  Each of these
needs to be installed with npm.  If I've done my job, you should
be able to install everything with just `npm install`.

    # require 'coffee-script/register'
    express      = require 'express'
    path         = require 'path'
    favicon      = require 'serve-favicon'
    logger       = require 'morgan'
    cookieParser = require 'cookie-parser'
    bodyParser   = require 'body-parser'

Next, include our routes.  We need to include each file with a route
in it.  A little bit later, we'll tell Express how to use these
routes. Note that, as expected, these packages are literate
coffeescript.

    routes = require './routes/index'
    users = require './routes/users'

Initialize Express

    app = express()
    port = process.env.port || 4000

Set up the view engine folder and tell the view the view engine to
work with jade.

    app.set 'views', path.join(__dirname, 'views')
    app.set 'view engine', 'jade'

Now we start installing middleware.

For favicons, uncomment the following line after placing your favicon
in /public

    # app.use favicon "#{__dirname}/public/favicon.ico"

Install logger, bodyParsers, cookieParser, and static pages

    app.use logger 'dev'
    app.use bodyParser.json()
    app.use bodyParser.urlencoded {extended: false}
    app.use cookieParser()

Express's static middleware enables the hosting of static resources.
We can use this to host static resources like scripts and styles.
We'll set static up so that any static resources will be accessed from
the "public" folder.

    app.use express.static path.join "#{__dirname}/public"

Next, we start setting up routes.

    # console.log users
    app.use '/', routes
    app.use '/users', users

We also set up a general 404 handler.

    app.use (req, res, next) ->
        err = new Error('Not Found')
        err.status = 404
        next err

We can also provide an error handler to Express that will output
stack traces if we're in a development environment.

    if app.get 'env' is 'development'
        app.use (err, req, res, next) ->
            res.status (err.status or 500)
            res.render 'error',
                message: err.message
                error: err

Start the server.

    server = app.listen port
    console.log "Listening on port #{port}"
