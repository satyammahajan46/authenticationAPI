# Authentication API

This project was generated with nodejs v12.18.2

## Development server

Run `node server.js` for a dev server. Navigate to `http://localhost:3000/`. The app will not automatically reload if you change any of the source files, please use nodemon as dependency for that.

## How to use locally 
* Navigate to code in cmd and run npm install to install node packages


## Technical Specifications
* RESTful API
* Authentication using JWT(json web token)
* JavaScript

## 👩‍🎤 Configuring API for your need

1. **Use own mongoose ATLAS**

Head over to (https://www.mongodb.com/cloud/atlas) and create your new account. Once you have an account, create a new cluster for connection URI.

2. **Add the URI to your new project**

In your `app.js` file, add your connection uri to the `MONGOSSE_CONNECTION_URI` field:

```
mongoose
  .connect(
    // Use your OWN uri or use envirnment variable
    `MONGOSSE_CONNECTION_URI`,
    { useFindAndModify: false }
  )
  .then((result) => {
    console.log("Connection established with database");
  })
  .catch((err) => console.log(err));
```
3. **Define your Custom Types**

This starter uses sample user model:

Customize sample models as required

**1. user.js** (USER model)


```
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

//Sample Model edit accordingly
const userSchema = new Schema(
//Add extra fields required
  {
    email: {
      type: String,
      required: true,
    },
    password: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    quiz: [
      {
        type: Schema.Types.ObjectId,
        ref: "Quiz",
      },
    ],
  },
  { timestamps: true }
);

module.exports = mongoose.model("User", userSchema);
```

**2. Validation** (For every extra field added to user.js schema, add required validation in routes/auth.js)
```
const express = require("express");
const { body } = require("express-validator");

const User = require("../models/user");
const authController = require("../controllers/auth");

const router = express.Router();

// Add validation if extra fields are added
router.post(
  "/signup",
  [
  //ADD_HERE don't forget the , after :)
    body("email")
      .isEmail()
      .withMessage("Please enter a valid email.")
      .custom((value, { req }) => {
        return User.findOne({ email: value }).then((userDoc) => {
          if (userDoc) {
            return Promise.reject("E-Mail address already exists!");
          }
        });
      })
      .normalizeEmail(),
    body("password")
      .trim()
      .isLength({ min: 8 })
      .withMessage("Minimum length is 8"),
    body("cPassword")
      .trim()
      .custom((value, { req }) => {
        if (value === req.body.password) {
          return value;
        } else {
          throw new Error("Password don't match");
        }
      }),
    body("name").trim().not().isEmpty(),
  ],
  authController.signup
);

// Add validation if extra fields are added
router.post(
  "/login",
  [
    body("email")
      .isEmail()
      .withMessage("Please enter a valid email")
      .normalizeEmail(),
    body("password").trim().isLength({ min: 8 }),
  ],
  authController.login
);

module.exports = router;

```

**3. Controller** (Sample code is provided for signup and login in controller/auth.js, modify as per need)
```
const { validationResult } = require("express-validator/check");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

const User = require("../models/user");

//Sample code for signup for necessary modifications
exports.signup = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    const error = new Error("Validation failed.");
    error.statusCode = 422;
    error.data = errors.array();
    throw error;
  }
  const email = req.body.email;
  const name = req.body.name;
  const password = req.body.password;
  bcrypt
    .hash(password, 12)
    .then((hashedPw) => {
      const user = new User({
        email: email,
        password: hashedPw,
        name: name,
      });
      return user.save();
    })
    .then((result) => {
      res.status(201).json({ message: "User created!", userId: result._id });
    })
    .catch((err) => {
      if (!err.statusCode) {
        err.statusCode = 500;
      }
      next(err);
    });
};

//Sample code for login use necessary modification
exports.login = (req, res, next) => {
  const email = req.body.email;
  const password = req.body.password;
  let loadedUser;
  User.findOne({ email: email })
    .then((user) => {
      if (!user) {
        const error = new Error("A user with this email could not be found.");
        error.statusCode = 401;
        throw error;
      }
      loadedUser = user;
      return bcrypt.compare(password, user.password);
    })
    .then((isEqual) => {
      if (!isEqual) {
        const error = new Error("Wrong password!");
        error.statusCode = 401;
        throw error;
      }
      //Expires in 1hr
      const expiresIn = new Date(Date.now() + 3600000);

      const token = jwt.sign(
        {
          email: loadedUser.email,
          userId: loadedUser._id.toString(),
        },
        "ENTER_YOUR_SECRET_KEY",
        { expiresIn: "1h" }
      );

      res.status(200).json({
        token: token,
        userId: loadedUser._id.toString(),
        expiresIn: expiresIn.toISOString(),
      });
    })
    .catch((err) => {
      if (!err.statusCode) {
        err.statusCode = 500;
      }
      next(err);
    });
};

```
**4. Add middleware for protected routes** (for every route you want to protect add middleware example below
```

const express = require("express");
const { body } = require("express-validator/check");

const postsController = require("../controllers/posts");
const isAuth = require("../middleware/is-auth");

const router = express.Router();

// GET request 
router.get("/sample", isAuth, postsController.getAllPosts);

// post request 
router.post("/create-post", isAuth, ADD_VALIDATION_HERE ,postsController.createPost);

module.exports = router;


```


# Tips & Common issues:

Thank you to everyone for contributing!

Before running server.js, make sure to install following dependency: jsonwebtoken, mongoose, express, express-validator, bcryptjs 

Additionally, clearing the cache, node modules, and package-lock.json can also clear your slate. 
1. `rm -rf node_modules .cache package-lock.json`
2. `npm install`


