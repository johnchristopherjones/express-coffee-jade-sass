In this route, we define an example route for a users API.

As a route, we need access to express and express' router.

    express = require 'express'
    router = express.Router()

A basic API should have a simple set of routes that use the
standard HTTP methods.  Every route in this file starts with /users
because of how it is included in app.coffee.md.

## Placeholder
Since this demonstration doesn't implement any file writing, we
initialize a dummy object.

    users =
        1000: "Alice"
        2000: "Bob"
        3000: "Charlie"

## GET
Get all users.  GET is **idempotent**.

    router.get '/', (req, res) ->
        res.send users

## POST
Create users.

    router.post '/', (req, res) ->
        for key, value of req.body
            users[key] = value
        res.send req.body

## GET :id_user
Get a single user by user id.  GET is **idempotent**.

    router.get ':id_user', (req, res) ->
        res.send users[req.params.id_user]

## PUT :id_user
Update a single user.  PUT is **idempotent**.

    router.put ':id_user', (req, res) ->
        users[req.params.id_user] = req.body
        res.send req.body

## DELETE :id_user
Delete a single user.  DELETE is **idempotent**.

    router.delete ':id_user', (req, res) ->
        id_user = req.params.id_user
        user = users[id_user]
        delete users[id_user]
        res.send {messsage: "User #{id_user} deleted."}

    module.exports = router
