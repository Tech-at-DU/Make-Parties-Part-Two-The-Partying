# Make Parities Part-Two: The-Partying

This is a challenge that builds on top of the [Make Parties tutorial](https://www.makeschool.com/academy/track/make-tweets). Please complete that tutorial before beginning this one. 

This is an "advanced tutorial" in that any step that you already know how to do by analogy with Make Parties you will be given no code, but any new steps you will be given code or links to instructions for the code.

### What You Will Learn

1. Signing Up and logging in with email & password (Authentication)

# 1. Creating Routes and Templates

We'll do an outside-in strategy where we build templates first and then controllers and then the models and migrations.

1. Create a `/sign-up` and `/login` links in the navbar.
2. Create the GET `/sign-up` route and template, and GET `/login` route and template—put them in a new controller called `auth.js`.
3. Make the sign up and login forms.
4. Make the POST routes for `/sign-up` and `/login` paths and verify that when you submit the forms the route code runs.


# 2. Creating User Model 

Now that we have the templates for sign up and login done, let's build the `User` model and table so we can create users.

1. Using sequelize-cli create a new model and migration for a model called User with the attributes `firstName`, `lastName`, `email`, and `password` all type `STRING`. We call the field `password` but we WILL NOT be saving someone's password to the database (that would be insecure and a violation of their privacy). Instead we will be saving a password "digest" or encrypted token of their password that through the magic of mathematics we can use to check the authenticity of future login attempts.
2. Run the migration and verify in postico that your model was created.
3. Create a user model file and add the model code. It should look something like this:

```js
const bcrypt = require('bcrypt')

'use strict';
module.exports = (sequelize, DataTypes) => {
  var User = sequelize.define('User', {
    firstName: DataTypes.STRING,
    lastName: DataTypes.STRING,
    email: Datatypes.STRING
    passwordDigest: DataTypes.STRING,
  })

  User.associate = function(models) {
  
  }

  return User;
};
```

4. Add the code to create a new user in the POST `/sign-up` route.
5. Test if you can create a new user. NOTICE that the password is NOT ENCRYPTED. This is unacceptable so we must add a step to encrypt this password.


# 3. Encrypting Passwords

In order to encrypt passwords when someone registers, we will use what is called a "model hook."

1. Add a "beforeCreate" hook to your User model.

```js
  User.hook('beforeCreate', async function(user) {
    const salt = await bcrypt.genSalt(10); //whatever number you want
    console.log(user);
    user.password = await bcrypt.hash(user.password, salt);
  })
```

2. npm install `bcrypt` the library we will use to encrypt passwords.
3. Let's now add the code to use bcrypt to intercept the password and encrypt it. To encrypt it we will be salting and then hashing the password.

```js
  User.hook('beforeCreate', async function(user) {
    const salt = await bcrypt.genSalt(10); //whatever number you want
    console.log(user);
    user.password = await bcrypt.hash(user.password, salt);
  })
```

To understand salting and hashing, watch this video that explains salting and hashing and how to hack passwords.

![Salting and Hashing](https://www.youtube.com/watch?v=--tnZMuoK3E)

# 4. Authenticating with JWT Token

### Generate JWT 

Now we need to be able to generate a secure JWT (JSON Web Token) after someone one signs up or logs in. We'll use the `jsonwebtoken` library to handle JWT tokens. Be sure to install `jsonwebtoken` in your project with npm.

Now add it to your `auth.js` controller and let's write a helper function that uses `jsonwebtoken` to create a JWT.

```js
// auth.js controller
const jwt = require('jsonwebtoken');

function generateJWT(user) {
  const sbarJWT = jwt.sign({ id: user._id, currentOrgId: user.orgs[0]._id }, "AUTH-SECRET", { expiresIn: 60*60*24*60 });

  return sbarJWT
}
```

Now to create the JWT token after a user is created in your POST `/sign-up` route. We call it mpJWT for "make parties JWT"

```js
// after creating the user
const mpJWT = generateJWT(user)
```

### Save JWT to Cookie 

Now that we have our JWT token we need to attach it to the client by using a cookie.

```js
// after creating the user
const mpJWT = generateJWT(user)

// save as cookie
res.cookie("mpJWT", mpJWT)

// redirect to the root route
res.redirect('/')
```

OMG your are technically logged in after you sign up now. But we won't have any functionality running off of this authentication yet, so let's get that working next before  tackling logging in.

# 5. Detecting if Someone is Logged in or Not

We're going to use some middleware to check if a cookie is present with a valid JWT token. If it is we'll add a `req.user` object to the request, kind like we added `req.body` when form data was present! The presence of this `req.user` object means that someone is logged in, and we can use the `req.user.id` attribute to look up their user record.

First install `express-jwt` and then initialize it in your initializations part of your `server.js` file. Then we'll use it in our middleware section.

```js
const jwtExpress = require('express-jwt');

// ...
// in your middleware inside your server.js file

app.use(jwtExpress({
    secret: "AUTH-SECRET,
    credentialsRequired: true,
    getToken: function fromHeaderOrQuerystring (req) {
      if (req.cookies.sbarJWT) {
        req.session.returnTo = null;
        return req.cookies.sbarJWT;
      }
      return null;
    }
  }).unless({ path: ['/', '/login', '/sign-up'] })
);
```

With this middleware above in place, if a valid JWT token is present in a cookie, then we will have a `req.user` object available in each controller route a logged  in user visits.

Let's test this by logging our `req.user` to the console in the root route.

```
// in root route
console.log(req.user)
```

We'll use the user's id from this `req.user` to look up the user's record in the future to do associations.

# 6. Creating a `currentUser` object 

Let's go one step further and get a `currentUser` object in memory that we can reference anywhere in our application, including in our templates. This will allow us to make our views look different for people who are logged in.

Let's add this custom middleware to your middleware in `server.js`.

```js
app.use(req, res, next => {
  // if a valid JWT token is present
  if (req.user) {
    // Look up the user's record
    User.findByPk(req.user.id, (currentUser) => {
      // make the user object available in all controllers and templates
      res.locals.currentUser = currentUser;
    });
  };
});
```

`res.locals` is a neat way to make any data available across all controllers and even inside of all views! 

# 7. Changing the Views For Logged In Users 

Once someone is logged in, they shouldn't see the Login and Sign Up links anymore. So let's wrap those in an if statement and only display them if the `currentUser` object is absent.

```html
{{#if currentUser}}
<!-- login and sign up links -->
{{else}}
<!-- link to my profile '/me' -->
{{/if}}
```

# 8. Testing 

So now when you are logged in (when a valid JWT is attached to a cookie in the client), you should not see the login and sign up links! Test your code manually and double check that it is working.


# 9. Logging In 

Now that we have sign up working, lets make login work. Logging in is like signing up, but we don't create a new user record, we check to see if the passwords match. 

First we have to add a `comparePassword` method to our `User` model.


```js
// user.js

User.comparePassword = function(password, done) {
  bcrypt.compare(password, this.password, function(err, isMatch) {
    return done(err, isMatch);
  });
};

```

Now we have to add our `/login` POST route that will look up the user by their email address, compare their passwords, and create a JWT token if their password is correct. **This is a lot of code, so read each comment and each line so you know what is happening step by step.**



```js
// auth.js 

// LOGIN (POST)
router.post('/login', (req, res, next) => {
  // look up user with email
  User.findOne({ email: req.body.email }).then(user => {
    // compare passwords
    user.comparePassword(req.body.password, function (err, isMatch) {
      // if not match send back to login
      if (!isMatch) {
        return res.redirect('/login');
      }
      // if is match generate JWT
      const mpJWT = generateJWT(user);
      // save jwt as cookie
      res.cookie("mpJWT", mpJWT)

      res.redirect('/')
    })
    .catch(err => {
      // if  can't find user return to login
      console.log(err)
      return res.redirect('/login');
    });
  });
});
```

Test logging in.


# 10. Logging Out 

Logging out is probably the simplest thing in the world because all we have to do is delete the JWT cookie and whamo, a usser is logged out.

```js
// auth.js controller 

// LOGOUT
router.get('/logout', (req, res, next) => {
  res.clearCookie('sbarJWT');

  req.session.sessionFlash = { type: 'success', message: 'Successfully logged out!' }
  return res.redirect('/');
});
```

Try logging in and logging out a few times.


# 11. Associating Resources with User

Now the fun stuff. If we have a user resource, we'll want to associate things they make with their user record. Remember that so long as you are logged in you have the `req.user` object which has an `req.user.id` attribute. So if we wanted to associate parties with users, we could add a `UserId` column to our `parties` table and then when we create a party, make sure to assign the currentUser's id to the party's UserId attribute.

```js
// parties.js controller

// create parties route 
party.UserId = req.user.id;
```

Also you'll need to add a has many parties association to the `User` model and a belongs to user assocation to the `Party` model. 

# More Permissions

So now you can be logged in or logged out and we can know who owns 

### Logged In User Permissions

What do you want people to be able to do when logged in?

Logged out users can:

1. See events
1. See Rsvps

Logged in users can:

1. Create events
1. RSVP

In order to protect these permissions use the same construction as above to only display buttons that you want visible to logged in people.

```html
{{#if currentUser}}
<!-- Things to display if user is logged in -->
{{/if}}
```

### Owners of Events (and Rsvps)

Do you want to make it so only people who created an event can edit it? 

Use a similar construction to see if someone owns an event (or an rsvp) to prevent non-owners from editing or deleting.

```html
{{#if event.userId = currentUser.id}}
<!-- buttons and links to edit and delete -->
{{/if}}
```
    
# 12. Associating RSVPs (challenge!)

Associate users with their RSVPs. 

# 13. Creating a User Profile (challenge!)

Now we can use our associated parties to create a user profile that can have our parties and our rsvps. 

Create GET request path at `/me` and load in the current user and their parties and rsvps.

List out the parities they've created and their rsvped parties.



# 14. Server-side Error Handling

No app is really done if it doesn't handle errors elegantly. There is no one right way to handle errors, so we'll learn one way that is very minimalistic and looks pretty nice. 

Error handling can be done on the front end (client-side) using JavaScript, but even when you have robust client-side error handling it is still a good idea to implement server-side error handling. The challenge with server-side error handling is our servers are RESTful meaning that each server endpoint has no knowledge of where a user has been before they arrive. So, how can we send a message from one route (where an error occurred) to another route (where the message will be displayed), e.g. over the form we just screwed up or a success message on our shiny new record we just successfully saved? The answer is *sessions*

### Adding Sessions

Install sessions and a cookie-parser using `npm i express-sessions cookie-parser`. Make sure to add a better secret.

```js
app.use(cookieParser("SECRET"));
const expiryDate = new Date(Date.now() + 60 * 60 * 1000 * 24 * 60) // 60 days

app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: {expires: expiryDate },
  store: sessionStore,
  resave: false
}));

```

### Adding Flash Alert HTML

Now add to your `main.handlebars` a little bootstrap alert block. This will appear only 

```html
{{#if sessionFlash && sessionFlash.message}}
<div class="alert fade show alert-dismissible alert-{{sessionFlash.type}}">
  {{sessionFlash.message}}
  <button.close(type='button', data-dismiss='alert', aria-label='Close')      
  <span (aria-hidden='true')>x</span>
</div>
{{/if}}
```

### Adding Custom Middleware

Now add some custom flash middleware to hand the flash message from the `req` to the `res` object.

```js
// Custom flash middleware -- from Ethan Brown's book, 'Web Development with Node & Express'
app.use(function(req, res, next){
    // if there's a flash message in the session request, make it available in the response, then delete it
    res.locals.sessionFlash = req.session.sessionFlash;
    delete req.session.sessionFlash;
    next();
});
```

### Adding the messgages

Now add the flash message commands where you want them.

```js
req.session.sessionFlash = { type: 'success', message: 'You will be sent an email with a link to change your password.' }
req.session.sessionFlash = { type: 'warning', message: 'No account exists associated with that email. Please try again.' }
```

# Test!







