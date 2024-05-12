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
