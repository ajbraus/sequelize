# Ultralight Implementation Guide

It is possible to get started with Sequelize very rapidly. This pattern uses Sequelize's `.sync()` method that builds and configures a SQL database on the fly. Using `.sync()` works best for experimenting, learning, and rapid prototyping, but is not recommended for production. For production releases review the [Standard Implementation Guide](manual/standard-guide.html).

The `.sync()` method builds a SQL database based on the structure of your sequelize models and associations. It has two important options:

* `.sync()` Drops db data and then creates tables and attributes if they do not exist.
* `.sync({ force: true })` Drops the entire database and recreates all tables and attributes.

**Working With Existing Databases**: If you are adding Sequelize to a project that has an existing SQL database, you may accelerate your implementation by using [Sequelize-Auto](https://github.com/sequelize/sequelize-auto) a library that will write Sequelize models based on your existing SQL tables.

## Single File Implementation

Sequelize can be fully implemented in a single JavaScript file. This is useful when building a microapp or when experimenting with or learning about ORMs, Sequelize, and SQL.

Give it a try!

Initialize a Node.js project and install sequelize.

```sh
$ npm init
$ npm i sequelize
```

Then add the following code to your root file, and run it.

```js
// CONNECT TO DB
const { Sequelize, Model, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

// DEFINE A MODEL
class User extends Model {}

User.init({
  username: DataTypes.STRING,
  birthday: DataTypes.DATE
}, { sequelize, modelName: 'user' });

(async () => {

  // BOOSTRAP DB WITH SYNC
  await sequelize.sync();

  // SAVE A RECORD
  const jane = await User.create({
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  });

  console.log(jane.toJSON());

})();
```

## Rapid Prototyping Implementation

Begin a Node.js project and install Sequelize, Sequelize-CLI, and your database adapter, in this case `sqlite3`.

```sh
$ npm init
$ npm i sequelize sequelize-cli sqlite3
```

Add a `.sequelizerc` file with the following configuration in order to setup a `db` folder for all sequelize-related files:

```
const path = require('path');
module.exports = {
  "config": path.resolve('./db/config', 'config.json'),
  "models-path": path.resolve('./db/models'),
  "seeders-path": path.resolve('./db/seeders'),
  "migrations-path": path.resolve('./db/migrations')
};
```

Now initialize Sequelize and generate .

```sh
$ sequelize init
```

Edit your `config/config.json` file to be prepared for production.

```json
{
 "production": {
   "use_env_variable": "DATABASE_URL"
 }
}
```

Create your first model. In this example we'll use a `Post` model, as if we were creating a simple blog.

```js
'use strict';

module.exports = (sequelize, DataTypes) => {
  const Post = sequelize.define('posts', {
    title: {
      type: DataTypes.STRING,
      allowNull: false
    },
    content: {
      type: DataTypes.TEXT
    }
  });

  return Post;
};
```

Now replace the `models/index.js` code with the following:

```js
'use strict';

const Sequelize = require('sequelize');
var env       = process.env.NODE_ENV || 'development';
var config    = require(__dirname + '/config/config.json')[env];
const db = {};

// OPEN CONNECTION
if (config.use_env_variable) { // PRODUCTION
  var sequelize = new Sequelize(process.env[config.use_env_variable]);
} else { // DEVELOPMENT
  var sequelize = new Sequelize("sqlite::memory");
}

// INITIALIZE MODELS
const Post = require('./models/post.js')(sequelize, Sequelize);

// DEFINE ASSOCIATIONS

// EXPORT CONNECTION AND MODELS
db.sequelize = sequelize
db.Sequelize = Sequelize
db.Post = Post

module.exports = db
```

Now to use our database, import the `models/index.js` file into the file at the root of your project, open the connection, and then . Initialize the project.

```js
const db = require('./db');
 db
   .sequelize
   .sync()
   .then(async () => {
     console.log('connected to db')

     // SAVE A RECORD
     const post = await Post.create({
       title: 'A Great Post',
       content: 'Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.'
     });

     console.log(post.toJSON());

   })
```

**Note**: Its important to note that `sync()` here, while building your database on the fly without migrations, will also drop and recreate your database every time you run this file. If you want test data to exist, either hard code its creation (as we have year with `post`) or use a seeder file.

You can build any number of models and associate them with this pattern; however, when you are ready to move to production, you will have to build migrations for each model as is outlined in the [Standard Implementation Guide](manual/standard-guide.md)