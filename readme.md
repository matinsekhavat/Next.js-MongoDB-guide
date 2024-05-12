### How to Install and Connect MongoDB to our Next.js Project

---

# Connect To MongoDB

- First we should install mongoose package from npm
  then we should make utils function to connect file to db

```javascript
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
import connectToDb from "db.js";
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

`request.cookies.get("")`
