## feathers-service-verify-reset
Adds user email verification and password reset capabilities to local
[`feathers-authentication`](http://docs.feathersjs.com/authentication/local.html).

Email addr verification and handling forgotten passwords are common features
these days. This package adds that functionality to [Feathersjs](http://docs.feathersjs.com/).

The optional transactional emails sent contain a link including a 30-char slug.
The slug has a configurable expiry delay. Emails may be sent for:

- Email addr verification when a new user is created.
- Resending a new email addr verification, e.g. previous verification email was lost or is expired.
- Successful user verification.
- Sending an email when the password is forgotten.
- Successful password reset for a forgotten password.

The server does not handle any interactions with the user.
Leaving it a pure API server, lets it be used with both native and browser clients.

[![Build Status](https://travis-ci.org/eddyystop/feathers-service-verify-reset.svg?branch=master)](https://travis-ci.org/eddyystop/feathers-service-verify-reset)
[![Coverage Status](https://coveralls.io/repos/github/eddyystop/feathers-service-verify-reset/badge.svg?branch=master)](https://coveralls.io/github/eddyystop/feathers-service-verify-reset?branch=master)

## Code Example

The folder `example` presents a full featured server/browser implementation
whose UI lets you try every API.  

### Server

Configure package in Feathersjs.

```javascript
const verifyReset = require('feathers-service-verify-reset').service;

module.exports = function () { // 'function' needed as we use 'this'
  const app = this;
  app.configure(authentication);
  app.configure(verifyReset({ emailer })); // NEW
  app.configure(user);
  app.configure(message);
};

function emailer(action, user, params, cb) {
  switch (action) {
    // resend (send another verification email), verify (email addr has been verified)
    // forgot (send forgot password email), reset (password has been reset)
  }
  cb(null);
}
```

An email to verify the user's email addr can be sent when user if created on the server,
e.g. `/src/services/user/hooks/index`:

```javascript
const verifyHooks = require('../../../hooks').verifyResetHooks;

exports.before = {
  // ...
  create: [
    auth.hashPassword(),
    verifyHooks.addVerification(), // set email addr verification info
  ],

exports.after = {
  // ...
  create: [
    hooks.remove('password'),
    emailVerification, // send email to verify the email addr
    verifyHooks.removeVerification(), // NEW
  ],
};

function emailVerification(hook, next) {
  // ...
  next(null, hook);
}
```

### Client

Client loads a wrapper for the package

```html
<script src=".../feathers-service-verify-reset/lib/client.js"></script>
```

and then uses convenient APIs.

```javascript
// Add a new user, using standard feathers users service.
// Then send a verification email with a link containing a slug.
users.create(user, (err, user) => {
  // ...
});

// Resend another email address verification email. New link, new slug.
verifyReset.resendVerify(email, (err, user) => {
  // ...
});

// Verify email address once user clicks link in the verification email.
verifyReset.verifySignUp(slug, (err, user) => {
  // ..
});

// Authenticate (sign in) user, requiring user to be verified.
verifyReset.authenticate(email, password, (err, user) => {
  // ...
});

// Send email for a forgotten password. Email contains a link with a slug.
verifyReset.sendResetPassword(email, (err, user) => {
  // ...
});

// Reset the new password once the user follows the link in the reset email
// and enters a new password.
verifyReset.saveResetPassword(slug, password, (err, user) => {
  // ...
});
```

### Routing

The client handles all interactions with the user.
Therefore the server must serve the client app when an email link is followed,
and the client must do some routing based on the path in the link.

Assume you have sent the email link:
`http://localhost:3030/socket/verify/12b827994bb59cacce47978567989e`

The server serves the client app on `/socket`:

```javascript
app.use('/', serveStatic(app.get('public')))
  // ...
  .use('/socket', (req, res) => {
    res.sendFile(path.resolve(__dirname, '..', 'public', 'socket.html'));
  })
```

The client routes itself based on the URL. A way primitive routing could be:

```javascript
const [leader, provider, action, slug] = window.location.pathname.split('/');

switch (action) {
  case 'verify':
    verifySignUp(slug);
    break;
  case 'reset':
    resetPassword(slug);
    break;
  default:
    // normal app startup
}
```

## Motivation

Email addr verification and handling forgotten passwords are common features
these days. This package adds that functionality to Feathersjs.

## Install package

Install [Nodejs](https://nodejs.org/en/).

Run `npm install feathers-service-verify-reset --save` in your project folder.

You can then require the utilities.

`/src` on GitHub contains the ES6 source.
It will run on Node 6+ without transpiling.


## Install and run example

`cd example`

`npm install`

`npm start`

Point browser to `localhost:3030/socket` for the socketio client,
to `localhost:3030/rest` for the rest client.

The two clients differ only in their how they configure `feathers-client`.

## API Reference

The following properties are added to `user` data:

- `isVerified` {Boolean} if user's email addr has been verified.
- `verifyToken` {String|null} token (slug) emailed for email addr verification.
- `verifyExpires` {Number|null} date-time when token expires.
- `resetToken` {String|null?} optional token (slug) emailed for password reset.
- `resetExpires` {Number|null?} date-time when token expires.

See Code Example section above.

See `example` folder for a fully functioning example.

## Tests

`npm run test:es6` to run tests with the existing ES5 transpiled code.

`npm test` to transpile to ES5 code and then run tests on Nodejs 6+.

`npm run cover` to run tests plus coverage.

## Contributors

- [eddyystop](https://github.com/eddyystop)

## License

MIT. See LICENSE.