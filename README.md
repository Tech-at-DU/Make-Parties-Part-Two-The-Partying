# Make Parities Part-Two: The-Partying

This is a challenge that builds on top of the [Make Parties tutorial](https://www.makeschool.com/academy/track/make-tweets). Please complete that tutorial before beginning this one. 

This is an "advanced tutorial" in that any step that you already know how to do by analogy with Make Parties you will be given no code, but any new steps you will be given code or links to instructions for the code.

### What You Will Learn

1. Signing Up and logging in with email & password (Authentication)

# 1. Creating Routes and Templates

We'll do an outside-in strategy where we build templates first and then controllers and then the models and migrations.

1. Create a `/sign-up` and `/login` links in the navbar.
2. Create the `/sign-up` route and template, and `/login` route and template.
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




