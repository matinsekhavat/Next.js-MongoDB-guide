### How to Install and Connect MongoDB to our Next.js Project

---

# 1.Connect To MongoDB

- First we should install mongoose package from npm
  then we should make utils function to connect file to db
  `configs>db.js`

```javascript
const mongoose = require("mongoose");

export default async function connectToDb() {
  try {
    if (mongoose.connections[0].readyState) {
      //true is once is already Connected
      return false;
    }
    await mongoose.connect("mongodb://127.0.0.1/my_database");
    console.log("Connected Successfully");
  } catch (error) {}
}
```

- then for use in function handler we should import our connectToDb and use in first line of handler

```javascript
import connectToDb from "./configs/db.js";
async function handler(req, res) {
  if (req.method !== "GET") {
    return false;
  }
  connectToDb();
}
```

---

# Create Schema

- Schema is a blueprint of our data , the algorithm to how our data Object should be Created for models we create folder call models
  `models/userModel.js`

```javascript
const schema = mongoose.Schema({
  username: {
    type: String,
    required: true,
  },
  email: {
    required: true,
    type: String,
  },
  password: {
    required: true,
    type: String,
    minLength: 8,
  },
});
// Capital&unplural
const usersModel = mongoose.models.User || mongoose.model("User", schema);
//if exist dont make it twice
export default usersModel;
```

- in our project every time we want to access to userModel we just import that
- in this example we create every single js file for each models
  `moongose.model()` (remmeber don`t add "s" at the end of the files => mongo in create models already add "s" at the end")

`  Mongoose automatically looks for the plural version of your model name. For example, if you use
const MyModel = mongoose.model('Ticket', mySchema);
Then MyModel will use the tickets collection, not the ticket collection. For more details read the model docs.`

---

### 1.Create Data in model (POST Request)

- for make first data fill in model
  first we export userModel
  which that was `const usersModel = mongoose.model("User", schema);`

  we import that in main function

  then say :

  ```javascript
  import userModel from "path";
  // for post and create and insert into model
  const user = await userModel.create({ username, email, password });
  if (user) {
    return res.status(201).json({ message: "create user successfuly", user });
  } else {
    return res.status(400).json({ message: "invalid user data" });
  }
  ```

---

### 2.Get all collections data (GET request)

for receive All data in user collection for EX:

```javascript
import userModel from "path";
const users = await userModel.find();
return res.status(200).json({ message: "Get All users successfuly", users });
```

---

### 3.get Single object from all collections data (single GET request)

- for catch one single object from model:
  notice for dev this function handler api we need dynamic path also
  cause we get id from req.query

`[id].js`

in handler function we get this params from:
`req.query`

```javascript
import userModel from "path";
const user = await userModel.findOne({ _id: req.query });
```

- you can also do this with find method
  **find vs findOne**
  if we search for 2 for example username
  find return array of object with all same searchs
  but findOne returns first object that find
  so for this case it's better to use findOne method

- search between to values while find

```javascript
const isUserExist = await UserModel.findOne({
  $or: [{ username }, { email }],
});
```

---

### 4.delete an item from all collections data (DELETE request)

for delete one single item from data base
`model.findOneAndDelete()`

```javascript
import userModel from "path";

const user = await userModel.findOneAndDelete({ _id: req.query });
```

---

### 5.update an item from all collections data (PUT request)

for delete one single item from data base
`model.findOneAndUpdate()`

```javascript
import userModel from "path";

const user = await userModel.findOneAndUpdate(
  { _id: req.query },
  { username, email, password }
);
```

---

### validate user ObjectID

````javascript
import { isValidObjectId } from "mongoose";

if (!isValidObjectId(req.query)) {
  const user = await userModel.findOneAndDelete({ _id: req.query });
}
```### How to use this collections in SSR & SSG

```javascript
import connectToDb from "path";
import usersModel from "path";
export async function getStaticProps() {
  connectToDb();
  const users = await usersModel.find();
  return {
    props: {
      users: JSON.parse(JSON.stringify(users)),
    },
  };
}
````

---

### How to use this collections in SSR & SSG

```javascript
import connectToDb from "path";
import usersModel from "path";
export async function getStaticProps() {
  connectToDb();
  const users = await usersModel.find();
  return {
    props: {
      users: JSON.parse(JSON.stringify(users)),
    },
  };
}
```

---

### timestamp & date

- for adding createdAt and updatedAt key and values

```javascript
const schema = mongoose.Schema({
  username: {
    type: String,
    required: true,
  },
  email: {
    required: true,
    type: String,
  },
  password: {
    required: true,
    type: String,
    minLength: 8,
  },
}, timestamps : true);
// Capital&unplural
const usersModel = mongoose.models.User || mongoose.model("User", schema);
//if exist dont make it twice
export default usersModel;
```

---

### make relation in MongoDB

<ul>
<li>One to One</li>
<li>One to Many</li>
<li>Many to Many</li>
</ul>

we use another key and value key is for example :
`teacher : ObjectID("")`

with this we make realtion
but how to write the schema?

```javascript
teacher : {
 type : mongoose.Types.ObjectId,
 ref:"Teacher"
 required :true
}
```

- here ref means this ObjectId is for which models?
- so in the ouptput we have problem we received datas from api like this
  `{teacher : "jdfbi475bdewb34i0"}`
- we dont want this so we want instaed of real id => shows us the {} that related to this key

##### for fix that : Populate()

```javascript
const courses = await coursesModel.find({}).populate("KEY in object");
```

for example here

```javascript
xyz : {
 type : mongoose.Types.ObjectId,
 ref:"Teacher"
 required :true
}

=====>
const courses = await coursesModel.find({}).populate("xyz");
//the key value in scheme object

```

in schema file also we should require teacherModel that we user theire data for relation in schema file

```javascript
const teacherModel = require("./teacherModel");
```

---

### select or remove specific fileds that comes from relations data

- sometime we dont need all keys & values
  wo how to delete one item or only get one key or 2 key ...

_if we want to say we need all data expect one of them we say "-price"_

```javascript
const users = await usersModle.find({}, "-__v0");
```

_if we want to say we need one of them we say "price email"_

```javascript
const users = await usersModle.find({}, "_id email");
or;
const courses = await coursesModel
  .find({}, "-price")
  .populate("xyz", "_id -updatedAt");
```

---

### kind of Relations in MongoDB

<ul>
<li>Refrence</li>
<li>Embedded</li>
</ul>
- in past examle we user ref : "" to ref that collection model
we have another way to make relation we called embedded

##### Embedded Relation

for make Embedded realtion when we use our schema we export that

```javascript
export const teacherSchema = mongoose.Schema({});

======>
const teacherModel = require("./teacherModel");
const courseSchema = mongoose.Schema({
  teacher: {
    type: teacherModel,
    required: true,
  },
})

```

but here we need all teacher schema structure
so first we get teached Schema

```javascript
const mainSchema = await teachersModel.findOne({ _id: req.body.id });
const courses = await courseModel.create({ email, teacher: mainSchema });
```

---

### virtual relation

- when we create our schema after that we can say
  virtual relation is all about optimization in Database so in genrall our relation is not created but when you sent req in our response we can see the final result

```javascript
const { commentsModel } = require("./comments");
schema.virtual("comments", {
  ref: "Comment",
  localField: "_id",
  foreignField: "course", // اون اسکیمایی که توش هستی
});
```

localfield is main model you are in there you say look at \_id in this file and connect to another model is foreignField we say "course"

when you want to recieve that you say

```javascript
const course = await courseModel
  .findOne({})
  .populate("name u enter in virtual || comments")
  .lean(); // return as array of obj
```

---

### validator package

- we can use validator package to validate our data
  `fastest-validator`

```javascript
const Validator = require("fastest-validator");
const v = new Validator();
const schema = {
  title: { type: "string", min: 8, max: 100 },
  price: { type: "number", minLength: 0, psitive: true },
};
const check = v.complie(schema);
export default check
====>
import courseValidator from "@/validators/course"
const validationResult = courseValidator(req.body)
if(validationResult !== true){
  return res.status(422).json(validationResult)
}
//if everything is alright return => true
//if not , we have problem while validating return {}
```

- so with this package our DB never crash

---

### work With MongoSH

- for make change and insert or get and ...inside the mongoCompass GUI

> for see how many database we already have :`show dbs`
> for select data base we say : `use <your-DataBase-Name>`

- so we in our database we have some collections
  we say : `db.<collection-name>.find({})`
  or
  `db.teachers.insertOne({SCHEMA})`

---

### Middlewares in next js

- security checking for exmaple everybody can not access to some private routes
  in this structure we `import NextResponse from "next/server"`
  we made `middleware.js` file in root of project

```javascript
import { NextResponse } from "next/server";
export function middleware(request) {
  if (request.nextUrl.pathname.startsWith("/users")) {
    //Validation if everything was ok so the NextResponse.next() will be execute
    return NextResponse.redirect(new URL("/login", request.url)); // dont return default so retun and redirect
  }
  return NextResponse.next(); //in default this code execute
}
```

we can also recieve users Cookis

## `request.cookies.get("")`

### matcher in MiddleWare

- we apple to this middlewares function execute in specific pages we want not into all pages

```javascript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// This function can be marked `async` if using `await` inside
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: "/about", // this middleware only execute when you are in /about routes
};

//for all routes in specific path
//matcher : "/panel/:path*"
//or array
// matcher : ["/user", "/panel"]
```

---

### .env files

.env files is more safe place to write forexmaple our ports or our apiKeys there

- 1.make `.env` files in root of project

```javascript
API_KEY = sdkndfkdfn;
```

- for access we say `proccess.env.API_KEY`

---

---

# 2.Authenticating in Backend

> packages we review in this document :

<ul>
  <li><a href="https://www.npmjs.com/package/bcryptjs">Hash-Password</a></li>
  <li><a href="https://www.npmjs.com/package/jsonwebtoken">JWT</a></li>
  <li><a href="https://www.npmjs.com/package/cookie">Cookies-manager</a></li>
</ul>
here we gonna to talk about all user auth and sign-in and sign-up
---

### JWT

- jwt is stand for JSON web Token , it's a library that helps us to make JWT we should not iclude neccessary data in token
  for example we can include user email to verify him with just email.
  jwt is made with 3 part :

1. Header
2. Payload
3. verify signature

---

### correct way to Authenticate user

in real Next.js project we need 3 Routes:

- routes (Auth)

1. Login (sign-in)
2. register(signup)
3. getme(get user info)
4. logout

---

### 1. sign-up

at first wo need create this path `api>auth` cause all of feature we want to add it's authenticate
then
create this files

- `api>auth>signin.js`
- `api>auth>signup.js`
- `api>auth>me.js`
- `api>auth>signout.js`

---

makes connection to Database :
`configs>db.js`

```javascript
const mongoose = require("mongoose");
async function connectToDB() {
  try {
    if (mongoose.connections[0].readyState) {
      // if already connected to db
      return false;
    } else {
      await mongoose.connect("databse-path/name");
      console.log("connect to DB successfuly");
    }
  } catch (err) {
    console.log("db connection has error ", err);
  }
}

export default connectToDB;
```

---

signup mechanism :
`models>User.js`
code in User.js :

```javascript
const mongoose = require("mongoose");

const schema = mongoose.Schema(
  {
    fisrtname: {
      type: String,
      required: true,
    },
    lastname: {
      type: String,
      required: true,
    },
    username: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: true,
    },
    password: {
      type: String,
      required: true,
    },
    role: {
      type:String
      required:true
    }
  },
  {
    timestamps: true,
  }
);
const model = mongoose.models.User || mongoose.model("User", schema);
export default model;
```

---

> so in sign-in progress we should check some important things :

- we should check if the user is already exist in the database or not so if the username or email exist we should give him a warn and prevent user to sign-up into our database
- we should validate user data that comes from client
  one importatnt thing is when we recieve user password , we should not display user password as un-hashable password we should always hash user password already in database.(learn in next lecture)
- then we should create one JWT for successful sign-up

```javascript
//we should also import utils file
import hashedPassword from "./utils/hashedPassword";
import generateToken from "./utils/generateToken";

export async function handler(req, res) {
  if (req.method !== "POST") {
    return null;
  }
  try {
    //connect to db
    connectToDb(); //should import this file in configs>db.js

    // get req.body
    const { fisrtname, lastname, username, email, password } = req.body;

    //simple validation
    if (
      !fisrtname.trim() ||
      !lastname.trim() ||
      !username.trim() ||
      !email.trim() ||
      !password.trim()
    ) {
      return res.status(422).json("data is Not valid");
    }

    //isUserExist?
    const isUserExist = await UserModel.findOne({
      $or: [{ username }, { email }],
    });

    //isUserExist return {with pair keys and values} ==> truthy value
    if (isUserExist) {
      return res.status(422).json({ message: "user is Already exist" });
    }

    //create

    //create & hash password & Token

    const hashedPassVar = await hashedPassword(password);
    const token = generateToken({ email });
    await UserModel.create({
      firstname,
      lastname,
      username,
      email,
      password: hashedPassVar,
      role: "USER",
    });

    //return
    return res.status(201).json({ message: "user create successfuly", token });

    //generate JWT
    //create
  } catch (error) {
    return res.status(500).json({ message: "unknown internal server error" });
  }
}
```

---

### hash password in database

- we should compile user password to unreadable password example:
  real password is : 7976@ali
  we use bcryptjs packages to save this in database like : "sfkndwkfkkefndkfnf".

  ***

  <a href="https://www.npmjs.com/package/bcryptjs" target="_blank">bcryptjs - package</a>

  1. npm i bcryptjs
  2. make a utils function to hash password => `utils>hashedPassword.js`

```javascript
import { hash } from "bcryptjs";

async function hashedPassword(pass) {
  // first argument is our password(string)
  // second argument is salt ==> default we set to : 12
  const hashedPass = await hash(pass, 12);
  return hashedPass;
}
export default hashedPassword;
```

---

### create JWT

- this step when user signup successfuly into website we want to create a token and set into his browser

1. so make a `utils>generateToken.js` file
2. we should install <a href="https://www.npmjs.com/package/jsonwebtoken">JSON Web Toekn</a> package.

_!NOTICE_ : for PrivateKey you should make a `.env` file and use your private Key inside that
(in this document we talked about what is .env files)
never use private key directly in the second argument of this sign Method

```javascript
import { sign } from "jsonwebtoken";
export function generateToken(data) {
  // first argument is all spread data we sent as a object
  // second argument is a PRIVATE_KEY as a JWT signature for better and high security
  // third argument is a some option.(for more details Read JWT Guideline)

  const token = sign({ ...data }, process.env.PRIVATE_KEY, {
    expiresIn: "24h",
  });
  //like : PRIVATE_KEY=knknpqzndbsqeuot
  return token;
}
```

---

### save token in cookies as HttpOnly

- it's better to back-end developr handle storing tokens in cookies in server and not sent cookis as a response to the client
- for this we use a package called <a href="https://www.npmjs.com/package/cookie">Cookies</a>

##### HTTTP Only

- HTTTP Only means we can not directly access to the token in cookies in the client side we always access to that in server for better security

1. npm i cookie

- so where we should set?

- when we want to `return res.json` to tell as response the signup user successfuly done
  but how?

```javascript
import { serialize } from "cookie";
return res
  .setHeader(
    "Set-Cookie",
    serialize("token", token, {
      httpOnly: true,
      path: "/",
      maxAge: 60 * 60 * 24, //  second
    })
  )
  .status(201)
  .json({ message: "user Created successfuly" });
```
