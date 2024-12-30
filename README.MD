# [TABLE OF CONTENTS](#table-of-contents)
- [TABLE OF CONTENTS](#table-of-contents)
- [Template Notes](#template-notes)
  - [Setup Notes](#setup-notes)
    - [`tsconfig.json`](#tsconfigjson)
    - [Configure the database with `db.config.ts`](#configure-the-database-with-dbconfigts)
    - [Initialize the database and ORM with `src/db/index.ts`](#initialize-the-database-and-orm-with-srcdbindexts)
    - [Define the ORM model with `models/tutorial.model.ts`](#define-the-orm-model-with-modelstutorialmodelts)
    - [Create the Repository with `src/repositories/tutorial.repository.ts`](#create-the-repository-with-srcrepositoriestutorialrepositoryts)
    - [Create the Controller with `src/controllers/tutorial.controller.ts`](#create-the-controller-with-srccontrollerstutorialcontrollerts)
    - [Create the Routes with `src/routes/index.ts` and `src/routes/tutorial.routes.ts`:](#create-the-routes-with-srcroutesindexts-and-srcroutestutorialroutests)

# Template Notes

These are my notes that are based on the tutorial and associated Github repository located here: https://www.bezkoder.com/typescript-orm-postgres / https://github.com/bezkoder/typescript-orm-postgres.  The tutorial provides the starting codebase that creates a server using NodeJS, Express, Sequelize (a promise-based NodeJS Object-Relational Mapping that supports dialects for Postgres, MySQL, SQL Server, etc.) and TypeScript.  Keep in mind the main goal of an ORM is to bridge the gap between the relational database and the programming language thereby providing a high-level abstraction layer to work with database entities as objects.  I will add notes in the order of the tutorial instructions and then review the codebase and provide a high level explanation of the architecture for my own understanding.  I hope to use this as a starting point to create my own TypeScript server that uses Sequelize to communicate with my Google Chrome Extension projects going forward.

## Setup Notes

Instead of following the tutorial verbatem I downloaded the sourcecode so all I need to do is run `npm install` to install `sequelize`, `sequelize-typescript`, `typescript`, `ts-node`, `@types/node` and `pg`.

[**Back to TOC**](#table-of-contents)

### `tsconfig.json`

The `tsconfig.json` file needs the following flags.  Since I cloned the sourcecode the `tsconfig.json` file has these flags preset and others are added as comments with explanations.
```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "es2016", /* Set the JavaScript language version for emitted JavaScript and include compatible library declarations. */
    "experimentalDecorators": true, /* Enable experimental support for legacy experimental decorators. */
    "emitDecoratorMetadata": true, /* Emit design-type metadata for decorated declarations in source files. */
    
    /* Modules */
    "module": "commonjs", /* Specify what module code is generated. */
    "resolveJsonModule": true, /* Enable importing .json files. */
    
    /* Emit */
    "outDir": "./build", /* Specify an output folder for all emitted files. */
    
    /* Interop Constraints */
    "esModuleInterop": true, /* Emit additional JavaScript to ease support for importing CommonJS modules. This enables 'allowSyntheticDefaultImports' for type compatibility. */
    "forceConsistentCasingInFileNames": true, /* Ensure that casing is correct in imports. */

    /* Type Checking */
    "strict": true, /* Enable all strict type-checking options. */

    /* Completeness */
    "skipLibCheck": true /* Skip type checking all .d.ts files. */
  }
}
```

[**Back to TOC**](#table-of-contents)

### Configure the database with `db.config.ts`

The `db.config.ts` file configures the Postgres database.  The `HOST`, `USER`, `PASSWORD`, `DB` and `pool` parameters are for PostgreSQL connection (`pool` is optional and will be used for Sequelize connection pool configuration).

- `max`: maximum number of connection in pool
- `min`: minimum number of connection in pool
- `idle`: maximum time, in milliseconds, that a connection can be idle before being released
- `acquire`: maximum time, in milliseconds, that pool will try to get connection before throwing error

```ts
export const config = {
  HOST: "localhost",
  USER: "postgres",
  PASSWORD: "123",
  DB: "testdb",
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
};

export const dialect = "postgres";
```

**Database Configuration In-Depth Explanation**

The `db.config.ts` file is used to configure the connection settings for your PostgreSQL database. This file exports two constants: `config` and `dialect`.

The `config` object contains the necessary parameters to establish a connection to the PostgreSQL database. The `HOST` parameter specifies the hostname of the database server, which in this case is set to `"localhost"`, indicating that the database server is running on the same machine as the application. The `USER` parameter is the username used to authenticate with the database, set to `"postgres"`. The `PASSWORD` parameter is the password for the specified user, set to `"123"`. The `DB` parameter specifies the name of the database to connect to, which is set to `"testdb"`.

Additionally, the `config` object includes a `pool` object that contains configuration settings for the connection pool. The `max` parameter sets the maximum number of connections in the pool to `5`. The `min` parameter sets the minimum number of connections in the pool to `0`. The `acquire` parameter specifies the maximum time (in milliseconds) that the pool will try to get a connection before throwing an error, set to `30000` (30 seconds). The `idle` parameter sets the maximum time (in milliseconds) that a connection can be idle before being released, set to `10000` (10 seconds).

The `dialect` constant specifies the type of database you are using. In this case, it is set to `"postgres"`, indicating that you are using a PostgreSQL database.

This configuration is used by the Sequelize ORM to connect to the PostgreSQL database and manage database connections efficiently.

[**Back to TOC**](#table-of-contents)

### Initialize the database and ORM with `src/db/index.ts`

Using the configuration above initialize the Sequelize ORM with the `src/db/index.ts`

```ts
import { Sequelize } from "sequelize-typescript";
import { config, dialect } from "../config/db.config";
import Tutorial from "../models/tutorial.model";

class Database {
  public sequelize: Sequelize | undefined;

  constructor() {
    this.connectToDatabase();
  }

  private async connectToDatabase() {
    this.sequelize = new Sequelize({
      database: config.DB,
      username: config.USER,
      password: config.PASSWORD,
      host: config.HOST,
      dialect: dialect,
      pool: {
        max: config.pool.max,
        min: config.pool.min,
        acquire: config.pool.acquire,
        idle: config.pool.idle
      },
      models: [Tutorial]
    });

    await this.sequelize
      .authenticate()
      .then(() => {
        console.log("Connection has been established successfully.");
      })
      .catch((err) => {
        console.error("Unable to connect to the Database:", err);
      });
  }
}

export default Database;
```

**Database Initialization and ORM Setup**

The `index.ts` file is responsible for initializing the database connection and setting up the Sequelize ORM (Object-Relational Mapping). This file imports the necessary modules and configuration settings, and defines a `Database` class that handles the connection to the PostgreSQL database.

The `Database` class contains a `sequelize` property, which is an instance of the `Sequelize` class from the `sequelize-typescript` library. The constructor of the `Database` class calls the `connectToDatabase` method to establish the connection.

The `connectToDatabase` method creates a new instance of `Sequelize` using the configuration settings imported from the `db.config.ts` file. These settings include the database name (`config.DB`), username (`config.USER`), password (`config.PASSWORD`), host (`config.HOST`), and dialect (`dialect`). Additionally, the method configures the connection pool with parameters such as `max` (maximum number of connections), `min` (minimum number of connections), `acquire` (maximum time to acquire a connection), and `idle` (maximum idle time for a connection).

The `models` property is set to an array containing the `Tutorial` model, which is imported from the `tutorial.model.ts` file. This allows Sequelize to manage the `Tutorial` table in the database.

The `connectToDatabase` method uses the `authenticate` method of the `Sequelize` instance to test the connection to the database. If the connection is successful, a message is logged to the console. If there is an error, it is caught and logged to the console.

[**Back to TOC**](#table-of-contents)

### Define the ORM model with `models/tutorial.model.ts`

This Sequelize Model represents the `tutorials` table within the Postgres database. These columns will be generated automatically: `id`, `title`, `description`, `published`.  Keep in mind that `createdAt` and `updatedAt` will be added by default.

After initializing Sequelize we don’t need to write CRUD functions because Sequelize supports all of the operations below which will be used in the `repository` folder's files:
- create a new Tutorial: `create(object)`
- find a Tutorial by id: `findByPk(id)`
- get all Tutorials: `findAll()`
- update a Tutorial by id: `update(data, where: { id: id })`
- remove a Tutorial: `destroy(where: { id: id })`
- remove all Tutorials: `destroy(where: {})`
- find all Tutorials by title: `findAll({ where: { title: ... } })`

`models/tutorial.model.ts`:
```ts
import { Model, Table, Column, DataType } from "sequelize-typescript";

@Table({
  tableName: "tutorials",
})
export default class Tutorial extends Model {
  @Column({
    type: DataType.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    field: "id"
  })
  id?: number;

  @Column({
    type: DataType.STRING(255),
    field: "title"
  })
  title?: string;

  @Column({
    type: DataType.STRING(255),
    field: "description"
  })
  description?: string;

  @Column({
    type: DataType.BOOLEAN,
    field: "published"
  })
  published?: boolean;
}

```

[**Back to TOC**](#table-of-contents)

**Explanation of `tutorial.model.ts`**

The `tutorial.model.ts` file defines the ORM (Object-Relational Mapping) model for the `Tutorial` entity using the `sequelize-typescript` library. This model represents the structure of the `tutorials` table in the PostgreSQL database and provides a way to interact with the table using TypeScript classes and objects.

The file begins by importing the necessary decorators and data types from the `sequelize-typescript` library. The `@Table` decorator is used to specify the table name in the database, which in this case is `"tutorials"`.

The `Tutorial` class extends the `Model` class provided by `sequelize-typescript`, which means it inherits all the functionality needed to interact with the database table. Each property of the `Tutorial` class is decorated with the `@Column` decorator to define the corresponding column in the database table.

The `id` property is defined as an integer column that serves as the primary key. It is set to auto-increment, meaning that its value will automatically increase with each new record inserted into the table. The `field` option specifies the actual column name in the database.

The `title` property is defined as a string column with a maximum length of 255 characters. The `description` property is also defined as a string column with a maximum length of 255 characters. The `published` property is defined as a boolean column.

By defining the `Tutorial` model in this way, Sequelize can map the properties of the class to the columns of the `tutorials` table in the database. This allows you to perform CRUD (Create, Read, Update, Delete) operations on the `tutorials` table using instances of the `Tutorial` class.

In the context of the server application architecture, the `Tutorial` model serves as a blueprint for the `tutorials` table in the database. It defines the structure of the table and provides methods to interact with the data. This abstraction allows you to work with database records as TypeScript objects, making it easier to manage and manipulate data within your application.

[**Back to TOC**](#table-of-contents)

**Explanation of TypeScript syntax and the sequelize-typescript library**

1. **imports from sequelize-typescript**

```ts
import { Model, Table, Column, DataType } from "sequelize-typescript";
```
- `Model`: This is the base class for all models in Sequelize. It provides methods to interact with the database.
- `Table`: This decorator is used to define the table name and other table-level options.
- `Column`: This decorator is used to define columns in the table.
- `DataType`: This is an enumeration of data types supported by Sequelize.

2. **the `@Table` decorator**
```ts
@Table({
  tableName: "tutorials",
})
```
- `@Table`: This decorator takes an object with table-level options. Here, the `tableName` option is set to `"tutorials"`, indicating that this model maps to the `tutorials` table in the database.

3. **The `Tutorial` class is defined and extends the `Model` class**
```ts
export default class Tutorial extends Model {
```
- `export default class Tutorial extends Model`: This defines the `Tutorial` class as the default export of the module which extends `Model`, meaning it inherits all the methods and properties of the `Model` class.  Each property of the `Tutorial` class is decorated with the `@Column` decorator to define the corresponding column in the database table.
```ts
  @Column({
    type: DataType.INTEGER,
    primaryKey: true,
    autoIncrement: true,
    field: "id"
  })
  id?: number;
```
- `@Column`: This decorator takes an object with column-level options.
    - `type`: Specifies the data type of the column. Here, it is set to `DataType.INTEGER`.
    - `primaryKey`: Indicates that this column is the primary key of the table.
    - `autoIncrement`: Specifies that the value of this column will automatically increment with each new record.
    - `field`: Specifies the actual column name in the database. Here, it is set to `"id"`.
- `id?: number`: This defines the id property as an optional number.

Similarly, other properties are defined:
```typescript
  @Column({
    type: DataType.STRING(255),
    field: "title"
  })
  title?: string;

  @Column({
    type: DataType.STRING(255),
    field: "description"
  })
  description?: string;

  @Column({
    type: DataType.BOOLEAN,
    field: "published"
  })
  published?: boolean;
```

- `title?: string`: This defines the `title` property as an optional string.
    - `type`: Specifies the data type of the column. Here, it is set to `DataType.STRING(255)`, indicating a string with a maximum length of 255 characters.
    - `field`: Specifies the actual column name in the database. Here, it is set to `"title"`.

- `description?: string`: This defines the `description` property as an optional string.
  - `type`: Specifies the data type of the column. Here, it is set to `DataType.STRING(255)`, indicating a string with a maximum length of 255 characters.
  - `field`: Specifies the actual column name in the database. Here, it is set to `"description"`.

- `published?: boolean`: This defines the `published` property as an optional boolean.
    - `type`: Specifies the data type of the column. Here, it is set to `DataType.BOOLEAN`.
    - `field`: Specifies the actual column name in the database. Here, it is set to `"published"`.

**Conclusion**
By using these decorators and TypeScript syntax, the `Tutorial` class defines the structure of the `tutorials` table and provides a way to interact with the table using TypeScript objects. This allows you to perform CRUD operations on the `tutorials` table in a type-safe manner.

[**Back to TOC**](#table-of-contents)

### Create the Repository with `src/repositories/tutorial.repository.ts`

The `tutorial.repository.ts` code below contains the following CRUD functions:
- save
- retrieveAll
- retrieveById
- update
- delete
- deleteAll

```ts
import { Op } from "sequelize";
import Tutorial from "../models/tutorial.model";

interface ITutorialRepository {
  save(tutorial: Tutorial): Promise<Tutorial>;
  retrieveAll(searchParams: {title: string, published: boolean}): Promise<Tutorial[]>;
  retrieveById(tutorialId: number): Promise<Tutorial | null>;
  update(tutorial: Tutorial): Promise<number>;
  delete(tutorialId: number): Promise<number>;
  deleteAll(): Promise<number>;
}

interface SearchCondition {
  [key: string]: any;
}

class TutorialRepository implements ITutorialRepository {
  async save(tutorial: Tutorial): Promise<Tutorial> {
    try {
      return await Tutorial.create({
        title: tutorial.title,
        description: tutorial.description,
        published: tutorial.published
      });
    } catch (err) {
      throw new Error("Failed to create Tutorial!");
    }
  }

  async retrieveAll(searchParams: {title?: string, published?: boolean}): Promise<Tutorial[]> {
    try {
      let condition: SearchCondition = {};

      if (searchParams?.published) condition.published = true;

      if (searchParams?.title)
        condition.title = { [Op.iLike]: `%${searchParams.title}%` };

      return await Tutorial.findAll({ where: condition });
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async retrieveById(tutorialId: number): Promise<Tutorial | null> {
    try {
      return await Tutorial.findByPk(tutorialId);
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async update(tutorial: Tutorial): Promise<number> {
    const { id, title, description, published } = tutorial;

    try {
      const affectedRows = await Tutorial.update(
        { title, description, published },
        { where: { id: id } }
      );

      return affectedRows[0];
    } catch (error) {
      throw new Error("Failed to update Tutorial!");
    }
  }

  async delete(tutorialId: number): Promise<number> {
    try {
      const affectedRows = await Tutorial.destroy({ where: { id: tutorialId } });

      return affectedRows;
    } catch (error) {
      throw new Error("Failed to delete Tutorial!");
    }
  }

  async deleteAll(): Promise<number> {
    try {
      return Tutorial.destroy({
        where: {},
        truncate: false
      });
    } catch (error) {
      throw new Error("Failed to delete Tutorials!");
    }
  }
}

export default new TutorialRepository();
```

[**Back to TOC**](#table-of-contents)

**Explanation of `tutorial.repository.ts`**

The `tutorial.repository.ts` file defines a repository class for managing CRUD (Create, Read, Update, Delete) operations on the `Tutorial` model. This repository pattern provides a layer of abstraction between the data access logic and the business logic of the application, making it easier to manage and test the code.

The file begins by importing the necessary modules and the Tutorial model:
```ts
import { Op } from "sequelize";
import Tutorial from "../models/tutorial.model";
```
The `Op` object from Sequelize is used to define operators for query conditions, such as `iLike` for case-insensitive pattern matching.

Next, the `ITutorialRepository` interface is defined to specify the methods that the repository class must implement:
```ts
interface ITutorialRepository {
  save(tutorial: Tutorial): Promise<Tutorial>;
  retrieveAll(searchParams: {title: string, published: boolean}): Promise<Tutorial[]>;
  retrieveById(tutorialId: number): Promise<Tutorial | null>;
  update(tutorial: Tutorial): Promise<number>;
  delete(tutorialId: number): Promise<number>;
  deleteAll(): Promise<number>;
}
```

This interface includes methods for saving a tutorial, retrieving all tutorials with optional search parameters, retrieving a tutorial by its ID, updating a tutorial, deleting a tutorial by its ID, and deleting all tutorials.

The `SearchCondition` interface is defined to represent the search conditions for querying the database:
```ts
interface SearchCondition {
  [key: string]: any;
}
```

The `TutorialRepository` class implements the `ITutorialRepository` interface and provides the actual implementation of the CRUD methods:
```ts
class TutorialRepository implements ITutorialRepository {
  async save(tutorial: Tutorial): Promise<Tutorial> {
    try {
      return await Tutorial.create({
        title: tutorial.title,
        description: tutorial.description,
        published: tutorial.published
      });
    } catch (err) {
      throw new Error("Failed to create Tutorial!");
    }
  }

  async retrieveAll(searchParams: {title?: string, published?: boolean}): Promise<Tutorial[]> {
    try {
      let condition: SearchCondition = {};

      if (searchParams?.published) condition.published = true;

      if (searchParams?.title)
        condition.title = { [Op.iLike]: `%${searchParams.title}%` };

      return await Tutorial.findAll({ where: condition });
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async retrieveById(tutorialId: number): Promise<Tutorial | null> {
    try {
      return await Tutorial.findByPk(tutorialId);
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async update(tutorial: Tutorial): Promise<number> {
    const { id, title, description, published } = tutorial;

    try {
      const affectedRows = await Tutorial.update(
        { title, description, published },
        { where: { id: id } }
      );

      return affectedRows[0];
    } catch (error) {
      throw new Error("Failed to update Tutorial!");
    }
  }

  async delete(tutorialId: number): Promise<number> {
    try {
      const affectedRows = await Tutorial.destroy({ where: { id: tutorialId } });

      return affectedRows;
    } catch (error) {
      throw new Error("Failed to delete Tutorial!");
    }
  }

  async deleteAll(): Promise<number> {
    try {
      return Tutorial.destroy({
        where: {},
        truncate: false
      });
    } catch (error) {
      throw new Error("Failed to delete Tutorials!");
    }
  }
}
```

- The `save` method creates a new tutorial record in the database using the `Tutorial.create` method.
- The `retrieveAll` method retrieves all tutorials that match the specified search parameters. It constructs a `condition` object based on the `title` and `published` parameters and uses the `Tutorial.findAll` method to query the database.
- The `retrieveById` method retrieves a tutorial by its primary key using the `Tutorial.findByPk` method.
- The `update` method updates an existing tutorial record in the database using the `Tutorial.update` method. It returns the number of affected rows.
- The `delete` method deletes a tutorial by its primary key using the `Tutorial.destroy` method. It returns the number of affected rows.
- The `deleteAll` method deletes all tutorials from the database using the `Tutorial.destroy` method with an empty `where` condition.

Finally, an instance of the `TutorialRepository` class is exported as the default export:
```ts
export default new TutorialRepository();
```

**Conclusion**
In the context of the server application architecture, the `TutorialRepository` class provides a centralized and consistent way to interact with the `Tutorial` model. It encapsulates the data access logic and ensures that the business logic remains clean and focused on the application's core functionality. This separation of concerns makes the code more maintainable and testable.

[**Back to TOC**](#table-of-contents)

**Explanation of TypeScript syntax and interface best practices**

The `tutorial.repository.ts` file defines a repository class for managing CRUD (Create, Read, Update, Delete) operations on the `Tutorial` model. This explanation will cover the TypeScript syntax used in the file, including how interfaces are defined and used in the `TutorialRepository` class.

**Import Statements**

1. First, the necessary modules and the `Tutorial` model are imported:
```ts
import { Op } from "sequelize";
import Tutorial from "../models/tutorial.model";
```
- `import { Op } from "sequelize";`: This imports the `Op` object from the `sequelize` library, which is used to define operators for query conditions.
- `import Tutorial from "../models/tutorial.model";`: This imports the `Tutorial` model from the specified path.

**ITutorialRepository interface**

2. Next, the `ITutorialRepository` interface defines the structure and contract for a repository that manages CRUD (Create, Read, Update, Delete) operations for the `Tutorial` model. Let's break down each line of the interface to understand its syntax and purpose.
```ts
interface ITutorialRepository {
  save(tutorial: Tutorial): Promise<Tutorial>;
  retrieveAll(searchParams: {title: string, published: boolean}): Promise<Tutorial[]>;
  retrieveById(tutorialId: number): Promise<Tutorial | null>;
  update(tutorial: Tutorial): Promise<number>;
  delete(tutorialId: number): Promise<number>;
  deleteAll(): Promise<number>;
}
```

**Line by Line Explanation**

```ts
interface ITutorialRepository
```

This line defines a new interface named `ITutorialRepository`. An **interface** in TypeScript is a way to define the shape of an object, specifying the properties and methods that the object must have. Interfaces are used to enforce a certain structure and ensure that any class implementing the interface adheres to this structure.  The following are its methods with syntax explained:

---

```ts
save(tutorial: Tutorial): Promise<Tutorial>;
```

This line defines a method named `save` within the `ITutorialRepository` interface.
- `save`: The name of the method.
- `(tutorial: Tutorial)`: The method takes a single parameter named `tutorial` of type `Tutorial`. The `Tutorial` type represents the model for a tutorial entity.
- `: Promise<Tutorial>`: The method returns a `Promise` that resolves to a `Tutorial` object. A `Promise` is an object representing the eventual completion or failure of an asynchronous operation. In this case, the `save` method will perform an asynchronous operation (such as saving the tutorial to the database) and return a `Promise` that resolves to the saved `Tutorial` object.

---

```ts
retrieveAll(searchParams: {title: string, published: boolean}): Promise<Tutorial[]>;
```

This line defines a method named `retrieveAll` within the `ITutorialRepository` interface.
- `retrieveAll`: The name of the method.
- `(searchParams: {title: string, published: boolean})`: The method takes a single parameter named `searchParams`, which is an object with two properties: `title` and `published`. Both properties are of type `string` and `boolean`, respectively.
- `: Promise<Tutorial[]>`: The method returns a `Promise` that resolves to an array of `Tutorial` objects. The `retrieveAll` method will perform an asynchronous operation (such as querying the database for tutorials that match the search parameters) and return a `Promise` that resolves to an array of matching `Tutorial` objects.

---

```ts
retrieveById(tutorialId: number): Promise<Tutorial | null>;
```
This line defines a method named `retrieveById` within the `ITutorialRepository` interface.
- `retrieveById`: The name of the method.
- `(tutorialId: number)`: The method takes a single parameter named `tutorialId` of type `number`. This parameter represents the unique identifier of the tutorial to be retrieved.
- `: Promise<Tutorial | null>`: The method returns a `Promise` that resolves to a `Tutorial` object or `null`. The `retrieveById` method will perform an asynchronous operation (such as querying the database for a tutorial with the specified ID) and return a `Promise` that resolves to the matching `Tutorial` object or `null` if no tutorial is found.

---

```ts
update(tutorial: Tutorial): Promise<number>;
```
This line defines a method named update within the ITutorialRepository interface.
- `update`: The name of the method.
- `(tutorial: Tutorial)`: The method takes a single parameter named `tutorial` of type `Tutorial`. This parameter represents the tutorial entity to be updated.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`. The `update` method will perform an asynchronous operation (such as updating the tutorial in the database) and return a `Promise` that resolves to the number of affected rows (indicating how many records were updated).

---

```ts
delete(tutorialId: number): Promise<number>;
```
This line defines a method named `delete` within the `ITutorialRepository` interface.
- `delete`: The name of the method.
- `(tutorialId: number)`: The method takes a single parameter named `tutorialId` of type `number`. This parameter represents the unique identifier of the tutorial to be deleted.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`. The `delete` method will perform an asynchronous operation (such as deleting the tutorial from the database) and return a `Promise` that resolves to the number of affected rows (indicating how many records were deleted).

---

```ts
deleteAll(): Promise<number>;
```
This line defines a method named `deleteAll` within the `ITutorialRepository` interface.
- `deleteAll`: The name of the method.
- `():`: The method does not take any parameters.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`. The `deleteAll` method will perform an asynchronous operation (such as deleting all tutorials from the database) and return a `Promise` that resolves to the number of affected rows (indicating how many records were deleted).

**Summary**
The `interface ITutorialRepository` defines a contract for a repository that manages CRUD operations for the `Tutorial` model. It specifies six methods: `save`, `retrieveAll`, `retrieveById`, `update`, `delete`, and `deleteAll`. Each method takes specific parameters and returns a `Promise` that resolves to the expected result. By defining this interface, we ensure that any class implementing it will provide the required methods with the specified signatures, promoting consistency and type safety in our code.

[**Back to TOC**](#table-of-contents)

---

3. The `SearchCondition` interface is defined to represent the search conditions for querying the database
```ts
interface SearchCondition {
  [key: string]: any;
}
```
- `interface SearchCondition`: This defines an interface named `SearchCondition`.
- `[key: string]: any;`: This is an index signature, which means that the `SearchCondition` object can have properties with any string key, and the values can be of any type.

4. The `TutorialRepository` class implements the `ITutorialRepository` interface and provides the actual implementation of the CRUD methods:
```ts
class TutorialRepository implements ITutorialRepository {
  async save(tutorial: Tutorial): Promise<Tutorial> {
    try {
      return await Tutorial.create({
        title: tutorial.title,
        description: tutorial.description,
        published: tutorial.published
      });
    } catch (err) {
      throw new Error("Failed to create Tutorial!");
    }
  }

  async retrieveAll(searchParams: {title?: string, published?: boolean}): Promise<Tutorial[]> {
    try {
      let condition: SearchCondition = {};

      if (searchParams?.published) condition.published = true;

      if (searchParams?.title)
        condition.title = { [Op.iLike]: `%${searchParams.title}%` };

      return await Tutorial.findAll({ where: condition });
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async retrieveById(tutorialId: number): Promise<Tutorial | null> {
    try {
      return await Tutorial.findByPk(tutorialId);
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }

  async update(tutorial: Tutorial): Promise<number> {
    const { id, title, description, published } = tutorial;

    try {
      const affectedRows = await Tutorial.update(
        { title, description, published },
        { where: { id: id } }
      );

      return affectedRows[0];
    } catch (error) {
      throw new Error("Failed to update Tutorial!");
    }
  }

  async delete(tutorialId: number): Promise<number> {
    try {
      const affectedRows = await Tutorial.destroy({ where: { id: tutorialId } });

      return affectedRows;
    } catch (error) {
      throw new Error("Failed to delete Tutorial!");
    }
  }

  async deleteAll(): Promise<number> {
    try {
      return Tutorial.destroy({
        where: {},
        truncate: false
      });
    } catch (error) {
      throw new Error("Failed to delete Tutorials!");
    }
  }
}
```

[**Back to TOC**](#table-of-contents)

**Detailed Explanation of the `TutorialRepository` Class**

The `TutorialRepository` class implements the `ITutorialRepository` interface and provides the actual implementation of the CRUD (Create, Read, Update, Delete) methods for the `Tutorial` model. Let's break down each method and line of code to understand how it works.

```ts
class TutorialRepository implements ITutorialRepository
```
This line defines a new class named `TutorialRepository` that implements the `ITutorialRepository` interface. By implementing the interface, the class must provide implementations for all the methods defined in the interface.

---

**`save` method**

The `save` method is responsible for creating a new tutorial record in the database. It takes a `Tutorial` object as a parameter and uses the `Tutorial.create` method to insert the new record into the `tutorials` table. If successful, it returns the created tutorial. If an error occurs, it throws an error message. This method is typically called by the `create` method in the `TutorialController` class when a new tutorial is created via an HTTP POST request.

```ts
  async save(tutorial: Tutorial): Promise<Tutorial> {
    try {
      return await Tutorial.create({
        title: tutorial.title,
        description: tutorial.description,
        published: tutorial.published
      });
    } catch (err) {
      throw new Error("Failed to create Tutorial!");
    }
  }
```

**Line by Line Explanation**
```ts
async save(tutorial: Tutorial): Promise<Tutorial>
```

This line defines an asynchronous method named `save` that takes a `Tutorial` object as a parameter and returns a `Promise` that resolves to a `Tutorial` object.
- `async`: The async keyword indicates that the method is asynchronous and will return a `Promise`.
- `tutorial: Tutorial`: The method takes a single parameter named `tutorial` of type `Tutorial`.
- `: Promise<Tutorial>`: The method returns a `Promise` that resolves to a `Tutorial` object.

---

```ts
try {
      return await Tutorial.create({
        title: tutorial.title,
        description: tutorial.description,
        published: tutorial.published
      });
    } catch (err) {
      throw new Error("Failed to create Tutorial!");
    }
```
The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is thrown.

---

```ts
return await Tutorial.create({ ... })
```
The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.create` method to resolve. This allows the code to be written in a synchronous style while still performing asynchronous operations.
- `Tutorial.create({ ... })`: This method creates a new record in the `tutorials` table with the specified properties (`title`, `description`, `published`).

---

```ts
catch (err) { throw new Error("Failed to create Tutorial!"); }
```
If an error occurs during the creation of the tutorial, an error message is thrown.

[**Back to TOC**](#table-of-contents)

**`retrieveAll` method**

The `retrieveAll` method retrieves all tutorials that match the specified search parameters. It takes an object with optional `title` and `published` properties as a parameter and constructs a search condition based on these properties. It then uses the `Tutorial.findAll` method to query the `tutorials` table and return the matching records. If an error occurs, it throws an error message. This method is typically called by the `findAll` and `findAllPublished` methods in the `TutorialController` class when tutorials are retrieved via HTTP GET requests.

```ts
  async retrieveAll(searchParams: {title?: string, published?: boolean}): Promise<Tutorial[]> {
    try {
      let condition: SearchCondition = {};

      if (searchParams?.published) condition.published = true;

      if (searchParams?.title)
        condition.title = { [Op.iLike]: `%${searchParams.title}%` };

      return await Tutorial.findAll({ where: condition });
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }
```

**Line by Line Explanation**
```ts
async retrieveAll(searchParams: {title?: string, published?: boolean}): Promise<Tutorial[]>
```
This line defines an asynchronous method named retrieveAll that takes an object with optional title and published properties as a parameter and returns a Promise that resolves to an array of Tutorial objects.
- `searchParams: {title?: string, published?: boolean}`: The method takes a single parameter named searchParams, which is an object with optional title and published properties.
- `: Promise<Tutorial[]>`: The method returns a Promise that resolves to an array of Tutorial objects.

---

```ts
let condition: SearchCondition = {};
```
This line initializes an empty object named `condition` of type `SearchCondition`. This object will be used to store the search conditions for querying the database.

---

```ts
if (searchParams?.published) condition.published = true;
```
If the `published` property is present in the `searchParams` object, it is added to the `condition` object.

---

```ts
if (searchParams?.title) condition.title = { [Op.iLike]: %${searchParams.title}% };
```
If the `title` property is present in the `searchParams` object, it is added to the `condition` object with a case-insensitive pattern matching operator (`iLike`).

---

```ts
return await Tutorial.findAll({ where: condition });
```
The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.findAll` method to resolve. This method retrieves all records from the `tutorials` table that match the specified conditions.

---

```ts
catch (error) { throw new Error("Failed to retrieve Tutorials!"); }
```
If an error occurs during the retrieval of tutorials, an error message is thrown.

---

**`retrieveById` method**

The `retrieveById` method retrieves a single tutorial by its ID. It takes a `tutorialId` as a parameter and uses the `Tutorial.findByPk` method to query the `tutorials` table and return the matching record. If an error occurs, it throws an error message. This method is typically called by the `findOne` method in the `TutorialController` class when a tutorial is retrieved by its ID via an HTTP GET request.

```ts
  async retrieveById(tutorialId: number): Promise<Tutorial | null> {
    try {
      return await Tutorial.findByPk(tutorialId);
    } catch (error) {
      throw new Error("Failed to retrieve Tutorials!");
    }
  }
```

**Line by Line Explanation**

```ts
async retrieveById(tutorialId: number): Promise<Tutorial | null>
```

This line defines an asynchronous method named `retrieveById` that takes a `tutorialId` as a parameter and returns a `Promise` that resolves to a `Tutorial` object or `null`.
- `tutorialId: number`: The method takes a single parameter named `tutorialId` of type `number`.
- `: Promise<Tutorial | null>`: The method returns a `Promise` that resolves to a `Tutorial` object or `null`.

[**Back to TOC**](#table-of-contents)

---

```ts
return await Tutorial.findByPk(tutorialId);
```

The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.findByPk` method to resolve. This method retrieves a record from the `tutorials` table by its primary key (`tutorialId`).

---

```ts
catch (error) { throw new Error("Failed to retrieve Tutorials!"); }
```

If an error occurs during the retrieval of the tutorial, an error message is thrown.

[**Back to TOC**](#table-of-contents)

---

**`update` method**

The `update` method updates an existing tutorial record in the database. It takes a `Tutorial` object as a parameter and uses the `Tutorial.update` method to update the record with the specified ID in the `tutorials` table. If successful, it returns the number of affected rows. If an error occurs, it throws an error message. This method is typically called by the `update` method in the `TutorialController` class when a tutorial is updated via an HTTP PUT request.

```ts
  async update(tutorial: Tutorial): Promise<number> {
    const { id, title, description, published } = tutorial;

    try {
      const affectedRows = await Tutorial.update(
        { title, description, published },
        { where: { id: id } }
      );

      return affectedRows[0];
    } catch (error) {
      throw new Error("Failed to update Tutorial!");
    }
  }
```

**Line by Line Explanation**

```ts
async update(tutorial: Tutorial): Promise<number>
```
This line defines an asynchronous method named `update` that takes a `Tutorial` object as a parameter and returns a `Promise` that resolves to a `number`.
- `tutorial: Tutorial`: The method takes a single parameter named `tutorial` of type `Tutorial`.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`.

---

```ts
const { id, title, description, published } = tutorial;
```
This line uses destructuring to extract the `id`, `title`, `description`, and `published` properties from the `tutorial` object.

---

```ts
const affectedRows = await Tutorial.update({ title, description, published }, { where: { id: id } });
```
The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.update` method to resolve. This method updates the specified properties of the record with the given `id` in the `tutorials` table.

---

```ts
return affectedRows[0];
```
The method returns the number of affected rows (indicating how many records were updated).

---

```ts
catch (error) { throw new Error("Failed to update Tutorial!"); }
```
If an error occurs during the update of the tutorial, an error message is thrown.

[**Back to TOC**](#table-of-contents)

---

**`delete` method**

The `delete` method deletes a single tutorial by its ID. It takes a `tutorialId` as a parameter and uses the `Tutorial.destroy` method to delete the record with the specified ID from the `tutorials` table. If successful, it returns the number of affected rows. If an error occurs, it throws an error message. This method is typically called by the `delete` method in the `TutorialController` class when a tutorial is deleted via an HTTP DELETE request.

```ts
  async delete(tutorialId: number): Promise<number> {
    try {
      const affectedRows = await Tutorial.destroy({ where: { id: tutorialId } });

      return affectedRows;
    } catch (error) {
      throw new Error("Failed to delete Tutorial!");
    }
  }
```

**Line by Line Explanation**
```ts
async delete(tutorialId: number): Promise<number>
```
This line defines an asynchronous method named `delete` that takes a `tutorialId` as a parameter and returns a `Promise` that resolves to a `number`.
- `tutorialId: number`: The method takes a single parameter named `tutorialId` of type `number`.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`.

---

```ts
const affectedRows = await Tutorial.destroy({ where: { id: tutorialId } });
```
The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.destroy` method to resolve. This method deletes the record with the given `id` from the `tutorials` table.

---

```ts
return affectedRows;
```
The method returns the number of affected rows (indicating how many records were deleted).

---

```ts
catch (error) { throw new Error("Failed to delete Tutorial!"); }
```
If an error occurs during the deletion of the tutorial, an error message is thrown.

[**Back to TOC**](#table-of-contents)

---

**`deleteAll` method**

The `deleteAll` method deletes all tutorials from the database. It uses the `Tutorial.destroy` method with an empty `where` condition to delete all records from the `tutorials` table. If successful, it returns the number of affected rows. If an error occurs, it throws an error message. This method is typically called by the `deleteAll` method in the `TutorialController` class when all tutorials are deleted via an HTTP DELETE request.

```ts
  async deleteAll(): Promise<number> {
    try {
      return Tutorial.destroy({
        where: {},
        truncate: false
      });
    } catch (error) {
      throw new Error("Failed to delete Tutorials!");
    }
  }
```

**Line by Line Explanation**
```ts
async deleteAll(): Promise<number>
```
This line defines an asynchronous method named `deleteAll` that returns a `Promise` that resolves to a `number`.
- `: Promise<number>`: The method returns a `Promise` that resolves to a `number`.

---

```ts
return Tutorial.destroy({ where: {}, truncate: false });
```
The `await` keyword is used to wait for the `Promise` returned by the `Tutorial.destroy` method to resolve. This method deletes all records from the `tutorials` table.

---

```ts
catch (error) { throw new Error("Failed to delete Tutorials!"); }
```
If an error occurs during the deletion of the tutorials, an error message is thrown.

---

**`TutorialRepository` class export**

Finally, an instance of the `TutorialRepository` class is exported as the default export:
```ts
export default new TutorialRepository();
```

- `export default new TutorialRepository();`: This creates a new instance of the `TutorialRepository` class and exports it as the default export of the module.

[**Back to TOC**](#table-of-contents)

---

**Summary of `TutorialRepository` Class methods**
The `TutorialRepository` class implements the `ITutorialRepository` interface and provides the actual implementation of the CRUD methods for the `Tutorial` model. Each method is defined as asynchronous using the `async` keyword and returns a `Promise`. The `await` keyword is used to wait for the `Promise` returned by the `Sequelize` methods to resolve, allowing the code to be written in a synchronous style while still performing asynchronous operations. The `try...catch` blocks are used to handle errors that may occur during the execution of the methods, ensuring that errors are caught and handled appropriately.

---

**TypeScript Best Practices**
1. **Use of Interfaces**: The `ITutorialRepository` interface defines the *contract* for the repository class, ensuring that the class implements all the required methods. This promotes consistency and makes the code easier to understand and maintain.

2. **Error Handling**: The `try...catch` blocks in the methods handle errors that may occur ***during database operations***. This ensures that errors are caught and handled appropriately, preventing the application from crashing.

3. **Asynchronous Operations**: The use of `async` and `await` allows the code to perform asynchronous operations in a synchronous style, making it easier to read and understand.

4. **Type Annotations**: The use of type annotations (e.g., `Promise<Tutorial>`, `Promise<number>`) provides type safety and helps catch errors at compile time.

By following these best practices, the `tutorial.repository.ts` file ensures that the repository class is well-defined, consistent, and easy to maintain.

[**Back to TOC**](#table-of-contents)

### Create the Controller with `src/controllers/tutorial.controller.ts`

The `TutorialController` class methods handle HTTP requests related to the `Tutorial` entity by using the `Tutorial` model and the `tutorialRepository` to perform CRUD operations. Each method is responsible for a specific operation, such as creating, retrieving, updating, or deleting tutorials, and sends appropriate HTTP responses based on the outcome of the operations. This approach follows the "controller, model, repository, and route" architecture, ensuring a clean separation of concerns and making the code more maintainable and testable.

Inside `src/controllers` folder, we create `tutorial.controller.ts` with these CRUD functions:
- create
- findAll
- findOne
- update
- delete
- deleteAll
- findAllPublished

```ts
import { Request, Response } from "express";
import Tutorial from "../models/tutorial.model";
import tutorialRepository from "../repositories/tutorial.repository";

export default class TutorialController {
  async create(req: Request, res: Response) {
    if (!req.body.title) {
      res.status(400).send({
        message: "Content can not be empty!"
      });
      return;
    }

    try {
      const tutorial: Tutorial = req.body;
      if (!tutorial.published) tutorial.published = false;

      const savedTutorial = await tutorialRepository.save(tutorial);

      res.status(201).send(savedTutorial);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }

  async findAll(req: Request, res: Response) {
    const title = typeof req.query.title === "string" ? req.query.title : "";

    try {
      const tutorials = await tutorialRepository.retrieveAll({ title });

      res.status(200).send(tutorials);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }

  async findOne(req: Request, res: Response) {
    const id: number = parseInt(req.params.id);

    try {
      const tutorial = await tutorialRepository.retrieveById(id);

      if (tutorial) res.status(200).send(tutorial);
      else
        res.status(404).send({
          message: `Cannot find Tutorial with id=${id}.`
        });
    } catch (err) {
      res.status(500).send({
        message: `Error retrieving Tutorial with id=${id}.`
      });
    }
  }

  async update(req: Request, res: Response) {
    let tutorial: Tutorial = req.body;
    tutorial.id = parseInt(req.params.id);

    try {
      const num = await tutorialRepository.update(tutorial);

      if (num == 1) {
        res.send({
          message: "Tutorial was updated successfully."
        });
      } else {
        res.send({
          message: `Cannot update Tutorial with id=${tutorial.id}. Maybe Tutorial was not found or req.body is empty!`
        });
      }
    } catch (err) {
      res.status(500).send({
        message: `Error updating Tutorial with id=${tutorial.id}.`
      });
    }
  }

  async delete(req: Request, res: Response) {
    const id: number = parseInt(req.params.id);

    try {
      const num = await tutorialRepository.delete(id);

      if (num == 1) {
        res.send({
          message: "Tutorial was deleted successfully!"
        });
      } else {
        res.send({
          message: `Cannot delete Tutorial with id=${id}. Maybe Tutorial was not found!`,
        });
      }
    } catch (err) {
      res.status(500).send({
        message: `Could not delete Tutorial with id==${id}.`
      });
    }
  }

  async deleteAll(req: Request, res: Response) {
    try {
      const num = await tutorialRepository.deleteAll();

      res.send({ message: `${num} Tutorials were deleted successfully!` });
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while removing all tutorials."
      });
    }
  }

  async findAllPublished(req: Request, res: Response) {
    try {
      const tutorials = await tutorialRepository.retrieveAll({ published: true });

      res.status(200).send(tutorials);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }
}
```

[**Back to TOC**](#table-of-contents)

---

**`tutorial.controller.ts` Explained**

The `TutorialController` class is responsible for handling HTTP requests related to the `Tutorial` entity. It uses the `Tutorial` model and the `tutorialRepository` to perform CRUD (Create, Read, Update, Delete) operations. Let's break down each method and line of code to understand how it works.

---

**Import Statements**
```ts
import { Request, Response } from "express";
import Tutorial from "../models/tutorial.model";
import tutorialRepository from "../repositories/tutorial.repository";
```

- `import { Request, Response } from "express";`: This imports the `Request` and `Response` types from the express library. These types are used to define the types of the `req` and `res` parameters in the controller methods.
- `import Tutorial from "../models/tutorial.model";`: This imports the `Tutorial` model from the specified path. The `Tutorial` model represents the structure of the `tutorials` table in the database.
- `import tutorialRepository from "../repositories/tutorial.repository";`: This imports the `tutorialRepository` instance from the specified path. The `tutorialRepository` provides methods to interact with the `tutorials` table in the database.

---

**Export Syntax**
```ts
export default class TutorialController {
```

This defines the TutorialController class as the default export of the module.

[**Back to TOC**](#table-of-contents)

---

**`create` method**

The `create` method handles the creation of a new tutorial. It first checks if the `title` property is present in the request body. If not, it sends a `400 Bad Request` response. If the `title` is present, it creates a `Tutorial` object from the request body and sets the `published` property to `false` if it is not provided. It then uses the `tutorialRepository.save` method to save the tutorial to the database. If successful, it sends a `201 Created` response with the saved tutorial. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async create(req: Request, res: Response) {
    if (!req.body.title) {
      res.status(400).send({
        message: "Content can not be empty!"
      });
      return;
    }

    try {
      const tutorial: Tutorial = req.body;
      if (!tutorial.published) tutorial.published = false;

      const savedTutorial = await tutorialRepository.save(tutorial);

      res.status(201).send(savedTutorial);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }
```

**Line by Line Explanation**
```ts
async create(req: Request, res: Response)
```
This line defines an asynchronous method named `create` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.
- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---
```ts
if (!req.body.title) {
  res.status(400).send({
    message: "Content can not be empty!"
  });
  return;
}
```
This block checks if the `title` property is present in the request body. If not, it sends a `400 Bad Request` response with an error message and returns from the method.

[**Back to TOC**](#table-of-contents)

---

```ts
try {
  const tutorial: Tutorial = req.body;
  if (!tutorial.published) tutorial.published = false;

  const savedTutorial = await tutorialRepository.save(tutorial);

  res.status(201).send(savedTutorial);
} catch (err) {
  res.status(500).send({
    message: "Some error occurred while retrieving tutorials."
  });
}
```
- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const tutorial: Tutorial = req.body;`: This line assigns the request body to a variable named `tutorial` of type `Tutorial`.
  - `if (!tutorial.published) tutorial.published = false;`: This line sets the `published` property to `false` if it is not present in the request body.
  - `const savedTutorial = await tutorialRepository.save(tutorial);`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.save` method to resolve. This method saves the tutorial to the database.
  - `res.status(201).send(savedTutorial);`: This line sends a `201 Created` response with the saved tutorial.
  - `catch (err) { res.status(500).send({ message: "Some error occurred while retrieving tutorials." }); }`: If an error occurs during the creation of the tutorial, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`findAll` method**

The `findAll` method retrieves all tutorials that match the `title` query parameter from the request. It extracts the `title` query parameter and uses the `tutorialRepository.retrieveAll` method to retrieve the tutorials from the database. If successful, it sends a `200 OK` response with the retrieved tutorials. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async findAll(req: Request, res: Response) {
    const title = typeof req.query.title === "string" ? req.query.title : "";

    try {
      const tutorials = await tutorialRepository.retrieveAll({ title });

      res.status(200).send(tutorials);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }
```

**Line by Line Explanation**
```ts
async findAll(req: Request, res: Response)
```

This line defines an asynchronous method named `findAll` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.

- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---

```ts
const title = typeof req.query.title === "string" ? req.query.title : "";
```

This line extracts the `title` query parameter from the request and assigns it to a variable named `title`. If the `title` query parameter is not a string, it assigns an empty string to the `title` variable.

---

```ts
try {
  const tutorials = await tutorialRepository.retrieveAll({ title });

  res.status(200).send(tutorials);
} catch (err) {
  res.status(500).send({
    message: "Some error occurred while retrieving tutorials."
  });
}
```

- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const tutorials = await tutorialRepository.retrieveAll({ title });`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.retrieveAll` method to resolve. This method retrieves all tutorials that match the specified title.
  - `res.status(200).send(tutorials);`: This line sends a `200 OK` response with the retrieved tutorials.
  - `catch (err) { res.status(500).send({ message: "Some error occurred while retrieving tutorials." }); }`: If an error occurs during the retrieval of tutorials, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`findOne` method**

The `findOne` method retrieves a single tutorial by its ID. It extracts the `id` parameter from the request URL and uses the `tutorialRepository.retrieveById` method to retrieve the tutorial from the database. If the tutorial is found, it sends a `200 OK` response with the retrieved tutorial. If the tutorial is not found, it sends a `404 Not Found` response. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async findOne(req: Request, res: Response) {
    const id: number = parseInt(req.params.id);

    try {
      const tutorial = await tutorialRepository.retrieveById(id);

      if (tutorial) res.status(200).send(tutorial);
      else
        res.status(404).send({
          message: `Cannot find Tutorial with id=${id}.`
        });
    } catch (err) {
      res.status(500).send({
        message: `Error retrieving Tutorial with id=${id}.`
      });
    }
  }
```

**Line by Line Explanation**

```ts
async findOne(req: Request, res: Response)
```

This line defines an asynchronous method named `findOne` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.
- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---

```ts
const id: number = parseInt(req.params.id);
```

This line extracts the `id` parameter from the request URL and assigns it to a variable named `id` of type `number`.

---

```ts
try {
  const tutorial = await tutorialRepository.retrieveById(id);

  if (tutorial) res.status(200).send(tutorial);
  else
    res.status(404).send({
      message: `Cannot find Tutorial with id=${id}.`
    });
} catch (err) {
  res.status(500).send({
    message: `Error retrieving Tutorial with id=${id}.`
  });
}
```
- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
    - `const tutorial = await tutorialRepository.retrieveById(id);`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.retrieveById` method to resolve. This method retrieves the tutorial with the specified ID.
    - `if (tutorial) res.status(200).send(tutorial);`: If the tutorial is found, a `200 OK` response is sent with the retrieved tutorial.
    - `else res.status(404).send({ message: Cannot find Tutorial with id=${id}. });`: If the tutorial is not found, a `404 Not Found` response is sent with an error message.
    - `catch (err) { res.status(500).send({ message: Error retrieving Tutorial with id=${id}. }); }`: If an error occurs during the retrieval of the tutorial, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`update` method**

The `update` method updates an existing tutorial by its ID. It creates a `Tutorial` object from the request body and sets the `id` property to the `id` parameter from the request URL. It then uses the `tutorialRepository.update` method to update the tutorial in the database. If the update is successful, it sends a success message. If the update is not successful, it sends an error message. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async update(req: Request, res: Response) {
    let tutorial: Tutorial = req.body;
    tutorial.id = parseInt(req.params.id);

    try {
      const num = await tutorialRepository.update(tutorial);

      if (num == 1) {
        res.send({
          message: "Tutorial was updated successfully."
        });
      } else {
        res.send({
          message: `Cannot update Tutorial with id=${tutorial.id}. Maybe Tutorial was not found or req.body is empty!`
        });
      }
    } catch (err) {
      res.status(500).send({
        message: `Error updating Tutorial with id=${tutorial.id}.`
      });
    }
  }
```

**Line by Line Explanation**

```ts
async update(req: Request, res: Response)
```

This line defines an asynchronous method named `update` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.
- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---

```ts
let tutorial: Tutorial = req.body;
tutorial.id = parseInt(req.params.id);
```
- `let tutorial: Tutorial = req.body;`: This line assigns the request body to a variable named `tutorial` of type `Tutorial`.
- `tutorial.id = parseInt(req.params.id);`: This line sets the `id` property of the `tutorial` object to the `id` parameter from the request URL.

---

```ts
try {
  const num = await tutorialRepository.update(tutorial);

  if (num == 1) {
    res.send({
      message: "Tutorial was updated successfully."
    });
  } else {
    res.send({
      message: `Cannot update Tutorial with id=${tutorial.id}. Maybe Tutorial was not found or req.body is empty!`
    });
  }
} catch (err) {
  res.status(500).send({
    message: `Error updating Tutorial with id=${tutorial.id}.`
  });
}
```
- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const num = await tutorialRepository.update(tutorial);`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.update` method to resolve. This method updates the tutorial in the database.
  - `if (num == 1) { res.send({ message: "Tutorial was updated successfully." }); }`: If the update is successful (i.e., one row is affected), a success message is sent in the response.
  - `else { res.send({ message: Cannot update Tutorial with id=${tutorial.id}. Maybe Tutorial was not found or req.body is empty! }); }`: If the update is not successful, an error message is sent in the response.
  - `catch (err) { res.status(500).send({ message: Error updating Tutorial with id=${tutorial.id}. }); }`: If an error occurs during the update of the tutorial, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`delete` method**

The `delete` method deletes a single tutorial by its ID. It extracts the `id` parameter from the request URL and uses the `tutorialRepository.delete` method to delete the tutorial from the database. If the deletion is successful, it sends a success message. If the deletion is not successful, it sends an error message. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async delete(req: Request, res: Response) {
    const id: number = parseInt(req.params.id);

    try {
      const num = await tutorialRepository.delete(id);

      if (num == 1) {
        res.send({
          message: "Tutorial was deleted successfully!"
        });
      } else {
        res.send({
          message: `Cannot delete Tutorial with id=${id}. Maybe Tutorial was not found!`,
        });
      }
    } catch (err) {
      res.status(500).send({
        message: `Could not delete Tutorial with id==${id}.`
      });
    }
  }
```

**Line by Line Explanation**
```ts
async delete(req: Request, res: Response)
```
This line defines an asynchronous method named `delete` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.
- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---

```ts
const id: number = parseInt(req.params.id);
```

This line extracts the `id` parameter from the request URL and assigns it to a variable named `id` of type `number`.

---

```ts
try {
  const num = await tutorialRepository.delete(id);

  if (num == 1) {
    res.send({
      message: "Tutorial was deleted successfully!"
    });
  } else {
    res.send({
      message: `Cannot delete Tutorial with id=${id}. Maybe Tutorial was not found!`,
    });
  }
} catch (err) {
  res.status(500).send({
    message: `Could not delete Tutorial with id==${id}.`
  });
}
```

- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const num = await tutorialRepository.delete(id);`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.delete` method to resolve. This method deletes the tutorial with the specified ID from the database.
  - `if (num == 1) { res.send({ message: "Tutorial was deleted successfully!" }); }`: If the deletion is successful (i.e., one row is affected), a success message is sent in the response.
  - `else { res.send({ message: Cannot delete Tutorial with id=${id}. Maybe Tutorial was not found! }); }`: If the deletion is not successful, an error message is sent in the response.
  - `catch (err) { res.status(500).send({ message: Could not delete Tutorial with id==${id}. }); }`: If an error occurs during the deletion of the tutorial, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`deleteAll` method**

The `deleteAll` method deletes all tutorials from the database. It uses the `tutorialRepository.deleteAll` method to delete all tutorials. If successful, it sends a success message with the number of deleted tutorials. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async deleteAll(req: Request, res: Response) {
    try {
      const num = await tutorialRepository.deleteAll();

      res.send({ message: `${num} Tutorials were deleted successfully!` });
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while removing all tutorials."
      });
    }
  }
```

**Line by Line Explanation**

```ts
async deleteAll(req: Request, res: Response)
```

This line defines an asynchronous method named `deleteAll` that takes a `Request` object (`req`) and a `Response` object (`res`) as parameters.
- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP response.

---

```ts
try {
  const num = await tutorialRepository.deleteAll();

  res.send({ message: `${num} Tutorials were deleted successfully!` });
} catch (err) {
  res.status(500).send({
    message: "Some error occurred while removing all tutorials."
  });
}
```

- `try { ... } catch (err) { ... }`: The `try...catch block` is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const num = await tutorialRepository.deleteAll();`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.deleteAll` method to resolve. This method deletes all tutorials from the database.
  - `res.send({ message: ${num} Tutorials were deleted successfully! });`: This line sends a success message with the number of deleted tutorials in the response.
  - `catch (err) { res.status(500).send({ message: "Some error occurred while removing all tutorials." }); }`: If an error occurs during the deletion of the tutorials, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

---

**`findAllPublished` method**

The `findAllPublished` method retrieves all published tutorials. It uses the `tutorialRepository.retrieveAll` method with the `published` property set to `true` to retrieve the tutorials from the database. If successful, it sends a `200 OK` response with the retrieved tutorials. If an error occurs, it sends a `500 Internal Server Error` response.

```ts
  async findAllPublished(req: Request, res: Response) {
    try {
      const tutorials = await tutorialRepository.retrieveAll({ published: true });

      res.status(200).send(tutorials);
    } catch (err) {
      res.status(500).send({
        message: "Some error occurred while retrieving tutorials."
      });
    }
  }
```

**Line by Line Explanation**
```ts
async findAllPublished(req: Request, res: Response)
```

This line defines an asynchronous method named findAllPublished that takes a Request object (req) and a Response object (res) as parameters.

- `async`: The `async` keyword indicates that the method is asynchronous and will return a `Promise`.
- `req: Request`: The method takes a single parameter named `req` of type `Request`. This parameter represents the HTTP request.
- `res: Response`: The method takes a single parameter named `res` of type `Response`. This parameter represents the HTTP resp

---

```ts
try {
  const tutorials = await tutorialRepository.retrieveAll({ published: true });

  res.status(200).send(tutorials);
} catch (err) {
  res.status(500).send({
    message: "Some error occurred while retrieving tutorials."
  });
}
```

- `try { ... } catch (err) { ... }`: The `try...catch` block is used to handle errors that may occur during the execution of the code inside the `try` block. If an error occurs, it is caught in the `catch` block, and an error message is sent in the response.
  - `const tutorials = await tutorialRepository.retrieveAll({ published: true });`: The `await` keyword is used to wait for the `Promise` returned by the `tutorialRepository.retrieveAll` method to resolve. This method retrieves all tutorials that are published (i.e., have the `published` property set to `true`).
  - `res.status(200).send(tutorials);`: This line sends a `200 OK` response with the retrieved tutorials.
  - `catch (err) { res.status(500).send({ message: "Some error occurred while retrieving tutorials." }); }`: If an error occurs during the retrieval of the tutorials, a `500 Internal Server Error` response is sent with an error message.

[**Back to TOC**](#table-of-contents)

**Summary of `TutorialController` Class Methods**

The `TutorialController` class handles HTTP requests related to the `Tutorial` entity. It uses the `Tutorial` model and the `tutorialRepository` to perform CRUD operations. Each method is defined as asynchronous using the `async` keyword and returns a `Promise`. The `await` keyword is used to wait for the `Promise` returned by the repository methods to resolve, allowing the code to be written in a synchronous style while still performing asynchronous operations. The `try...catch` blocks are used to handle errors that may occur during the execution of the methods, ensuring that errors are caught and handled appropriately.

**Conclusion**

In the context of the server application architecture, the `TutorialController` class provides a centralized and consistent way to handle HTTP requests related to the `Tutorial` entity. It encapsulates the request handling logic and ensures that the business logic remains clean and focused on the application's core functionality. This separation of concerns makes the code more maintainable and testable.

### Create the Routes with `src/routes/index.ts` and `src/routes/tutorial.routes.ts`:

In the context of the server application architecture, the `Routes` and `TutorialRoutes` classes provide a centralized and consistent way to set up the routes for handling HTTP requests related to the `Tutorial` entity. The `Routes` class initializes the main application routes, while the `TutorialRoutes` class sets up the routes for the tutorial-related operations. By mapping the routes to the corresponding methods of the `TutorialController` class, these classes ensure that the appropriate controller methods are called when HTTP requests are made to the tutorial-related endpoints. This separation of concerns makes the code more maintainable and testable.

**`src/routes/index.ts`**

The `index.ts` file defines the `Routes` class, which is responsible for setting up the main application routes. This class imports the necessary modules and initializes the routes for the application.

```ts
import { Application } from "express";
import tutorialRoutes from "./tutorial.routes";
import homeRoutes from "./home.routes";

export default class Routes {
  constructor(app: Application) {
    app.use("/api", homeRoutes);
    app.use("/api/tutorials", tutorialRoutes);
  }
}
```

**Line by Line Explanation**

1. Import Statements

```ts
import { Application } from "express";
import tutorialRoutes from "./tutorial.routes";
import homeRoutes from "./home.routes";
```
- `import { Application } from "express";`: This imports the `Application` type from the `express` library. The `Application` type represents an Express application.
- `import tutorialRoutes from "./tutorial.routes";`: This imports the `tutorialRoutes` instance from the `tutorial.routes.ts` file.
- `import homeRoutes from "./home.routes";`: This imports the `homeRoutes` instance from the `home.routes.ts` file.

2. Export Syntax

```ts
export default class Routes {
```

This line defines the `Routes` class as the default export of the module.

3. Constructor Syntax

The constructor of the `Routes` class initializes the routes for the application. It sets up the `homeRoutes` for the `/api` endpoint and the `tutorialRoutes` for the `/api/tutorials` endpoint. This ensures that the appropriate routes are used for handling HTTP requests related to the home and tutorial functionalities.

```ts
constructor(app: Application) {
  app.use("/api", homeRoutes);
  app.use("/api/tutorials", tutorialRoutes);
}
```

- `constructor(app: Application)`: This line defines the constructor of the `Routes` class, which takes an `Application` object (`app`) as a parameter.
  - `app.use("/api", homeRoutes);`: This line sets up the `homeRoutes` to be used for the `/api` endpoint.
  - `app.use("/api/tutorials", tutorialRoutes);`: This line sets up the `tutorialRoutes` to be used for the `/api/tutorials` endpoint.



**`src/routes/tutorial.routes.ts`**

The `tutorial.routes.ts` file defines the TutorialRoutes class, which is responsible for setting up the routes related to the Tutorial entity. This class imports the necessary modules and initializes the routes for the tutorial-related operations.

```ts
import { Router } from "express";
import TutorialController from "../controllers/tutorial.controller";

class TutorialRoutes {
  router = Router();
  controller = new TutorialController();

  constructor() {
    this.intializeRoutes();
  }

  intializeRoutes() {
    // Create a new Tutorial
    this.router.post("/", this.controller.create);

    // Retrieve all Tutorials
    this.router.get("/", this.controller.findAll);

    // Retrieve all published Tutorials
    this.router.get("/published", this.controller.findAllPublished);

    // Retrieve a single Tutorial with id
    this.router.get("/:id", this.controller.findOne);

    // Update a Tutorial with id
    this.router.put("/:id", this.controller.update);

    // Delete a Tutorial with id
    this.router.delete("/:id", this.controller.delete);

    // Delete all Tutorials
    this.router.delete("/", this.controller.deleteAll);
  }
}

export default new TutorialRoutes().router;
```

**Line by Line Explanation**

1. Import Statements
```ts
import { Router } from "express";
import TutorialController from "../controllers/tutorial.controller";
```
- `import { Router } from "express";`: This imports the `Router` class from the `express` library. The `Router` class is used to create modular, mountable route handlers.
- `import TutorialController from "../controllers/tutorial.controller";`: This imports the `TutorialController` class from the specified path. The `TutorialController` class contains the methods for handling HTTP requests related to the `Tutorial` entity.

---

2. TutorialRoutes class

```ts
class TutorialRoutes {
```

This line defines the `TutorialRoutes` class.

---

```ts
router = Router();
controller = new TutorialController();
```

- `router = Router();`: This line initializes a new `Router` instance and assigns it to the `router` property of the `TutorialRoutes` class.
- `controller = new TutorialController();`: This line initializes a new `TutorialController` instance and assigns it to the `controller` property of the `TutorialRoutes` class.

---

```ts
  constructor() {
    this.intializeRoutes();
  }
```

- `constructor()`: This line defines the constructor of the `TutorialRoutes` class.
  - `this.intializeRoutes();`: This line calls the `intializeRoutes` method to set up the routes for the tutorial-related operations.

---

```ts
  intializeRoutes() {
    // Create a new Tutorial
    this.router.post("/", this.controller.create);

    // Retrieve all Tutorials
    this.router.get("/", this.controller.findAll);

    // Retrieve all published Tutorials
    this.router.get("/published", this.controller.findAllPublished);

    // Retrieve a single Tutorial with id
    this.router.get("/:id", this.controller.findOne);

    // Update a Tutorial with id
    this.router.put("/:id", this.controller.update);

    // Delete a Tutorial with id
    this.router.delete("/:id", this.controller.delete);

    // Delete all Tutorials
    this.router.delete("/", this.controller.deleteAll);
  }
```

- `intializeRoutes()`: This line defines the `intializeRoutes` method, which sets up the routes for the tutorial-related operations.
  - `this.router.post("/", this.controller.create);`: This line sets up a POST route for creating a new tutorial. It maps the route to the `create` method of the `TutorialController` class.
  - `this.router.get("/", this.controller.findAll);`: This line sets up a GET route for retrieving all tutorials. It maps the route to the `findAll` method of the `TutorialController` class.
  - `this.router.get("/published", this.controller.findAllPublished);`: This line sets up a GET route for retrieving all published tutorials. It maps the route to the `findAllPublished` method of the `TutorialController` class.
  - `this.router.get("/:id", this.controller.findOne);`: This line sets up a GET route for retrieving a single tutorial by its ID. It maps the route to the `findOne` method of the `TutorialController` class.
  - `this.router.put("/:id", this.controller.update);`: This line sets up a PUT route for updating a tutorial by its ID. It maps the route to the `update` method of the `TutorialController` class.
  - `this.router.delete("/:id", this.controller.delete);`: This line sets up a DELETE route for deleting a tutorial by its ID. It maps the route to the `delete` method of the `TutorialController` class.
  - `this.router.delete("/", this.controller.deleteAll);`: This line sets up a DELETE route for deleting all tutorials. It maps the route to the `deleteAll` method of the `TutorialController` class.

---

```ts
export default new TutorialRoutes().router;
```

This line creates a new instance of the `TutorialRoutes` class and exports its `router` property as the default export of the module.

[**Back to TOC**](#table-of-contents)
