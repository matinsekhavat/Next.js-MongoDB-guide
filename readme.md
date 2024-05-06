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