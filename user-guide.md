# User Guide

1. [Deciding on a Backend Language](#backend-language)
2. [Use a Project Template](#use-a-project-template)
3. [Database Selection](#database-selection)
    1. [Docker](#docker)
    2. [Using MongoDB](#using-mongodb)
4. [Install Packages](#install-packages)
5. [Integrate Packages](#integrate-packages)
    1. [Python Backend](#python-backend)
    2. [NodeJS Backend](#nodejs-backend)
    3. [React Frontend](#react-frontend)

## Backend Language

The choice of backend language depends a lot on the additional libraries that you need for any computation and the skill
 set of the team working on a project. If you do not have any Python experience and don't require any Python libraries
 in your project, then don't choose it. If you have analytics / research code in Python however, your backend should be
 written in Python, so you can directly call the analytics / research code from the backend, unless you already have
 plans to run the analytics code in containers through some other platform (e.g. Orbit, WML, Castor), which can be
 integrated into a JavaScript backend as well.

If you require any other language other than JavaScript or Python for your backend, then you can still use the Frappy
 components in a separate application that manages the database content (content, data, results), but doesn't deliver
 the API / functionality required for your project. Your project can then connect to the same database and access the
 data managed by the Frappy app.

_Note that this guide uses "NodeJS" and "JavaScript" in the same context. NodeJS is a command line program interpreting
 JavaScript code._

**Important:** All Python packages require Python >= 3.6 - 2.x or 3.4 are not fully supported!

## Use a Project Template

The following 2 project templates are available:

- [NodeJS template](https://github.com/ilfrich/frappy-node-react-template)
- [Python template](https://github.com/ilfrich/frappy-flask-react-template)

From the Github pages linked above, you can directly choose "Use this template" (a green button next to the clone
 option). This allows you to use that code in your own repository.

Both project templates provide a README file with detailed instructions how to install any dependencies and how to use
 it. You should definitely:

- Adjust the README to describe your project - only retaining the sections about system requirements, installation and
 usage
- Clean up the code:
    - removing example API and stores
    - removing methods and code for the database that you are **NOT** using
    - adjust meta information of the project (e.g. for the NodeJS template adjust the `name` and `description` in the
     package.json)

After creating your own repository, it is time to choose your database.

## Database Selection

_MySQL is currently not yet supported - in development_

**General Considerations**

The choice of database depends a lot on the anticipated structure of the data required by your application. In general
 we recommend to use document-based databases, such as **MongoDB** as they are very flexible and easy to use. Relational
 databases, such as MySQL, are useful in situations where you have a lot of same-structured and low-complexity items to
 store, where the ability to query, sum up, count, etc. is more important than just simple data retrieval.

For web applications, usage of MySQL will cause additional effort to transform between rows of data (the structure used
 to store the data) and JSON (the structure used to transfer the data to the front-end or other applications). MongoDB
 stores JSON natively and therefore no conversion is required.

### Docker

In your local environment or on a demo server, you can run the database server as Docker container. Only install the
 database server on your machine, if you know what you're doing.

This repository contains Docker Compose files that can spin up a database for you in the
 [resources](https://github.com/ilfrich/frappy-documentation/tree/master/resources) folder. Simply download the file
 to your local (you can include it into your custom project repository) and run:

- **MongoDB**: `docker-compose -f docker-compose-mongo.yml up` - will start
 `mongodb://frappy:frappy@localhost:27018`. This is the `MONGO_URL` environment variable value you need to provide in
 the JavaScript and Python backend.

### Using MongoDB

The Frappy libraries wrap the MongoDB driver into store classes. These store classes can be generic
 (`AbstractMongoStore`) or specific to a model (`DataStore`, `ContentStore`, `UserStore`, ...).

- A MongoDB server can have multiple databases, where one database is usually used by one application.
- A MongoDB database can have multiple collections, where each collection stores a different type of object
    - _Collection names are usually provided in camel-case with lower-case first letter (JavaScript style)_

**Python Example**

```python
# import store class from package
from frappymongodata import DataStore

# instantiate store class
store = DataStore(mongo_url="mongodb://localhost:27017", mongo_db="myDbName", collection_name="dataSets")
# access methods of the store
all_data_sets = store.get_all()
```

**JavaScript Example**

```javascript
import mongodb from "mongodb"
import { DataSetStore } from "@frappy/node-mongo-dataset-store"

mongodb.MongoClient.connect("mongodb://localhost:27017", { useNewUrlParser: true }).then(client => {
    const store = new DataSetStore(client, "myDbName", "dataSets")  // client, dbName, collectionName
    store.getAll().then(allDataSets => {
        // ... do something with a list of data sets
    })
})
```

## Install Packages

NPM packages are name-scoped (using the `@frappy/<package_name>` prefix).

A list of all available packages can be found in the [main README](README.md#packages).

Packages exist at 3 different levels:

- Database packages (pip and npm)
- Backend packages (pip and npm)
- Frontend packages (npm)

The frontend packages are integrated tightly with the backend packages as they call the API provided by the backend
 packages. The backend packages depend highly on the methods provided by the database packages.

Each backend and frontend package's components support an `apiPrefix` (JavaScript) or `api_prefix` (Python) parameter
 that needs to match to connect the 2 components. If you omit those parameters, they will use the defaults that match up
 already. However, if you decide to change one (e.g. frontend package for authentication), you also need to change the
 counterpart (backend package for authentication respectively).

The frontend is always written in React (no choice here). The backend can be either Python or NodeJS. Packages for both
 choices are provided. The backend packages integrate into database packages provided for MongoDB and/or MySQL in both
 languages (Python and NodeJS).

```bash
# for JavaScript packages
npm install -S @frappy/node-authentication

# for Python packages (should also add this to the requirements.txt of your project)
pip install frappyflaskauth --user
```

## Integrate Packages

This is the final step. At this point you should have:

- a repository containing your application code base (potentially using the project templates)
- a database server running
- the runtime for your backend language installed and configured as well as npm/NodeJS (for the frontend)
- required packages for storage, backend and frontend installed in their respective components (backend / frontend)

Each package README provides detailed description of all parameters and available functions. This guide focuses more on
 the big picture and how individual packages fit together.

### Python Backend

The following example is using ALL of the Frappy packages to provide the full functionality using MongoDB for storage.
 Detailed descriptions and documentation are provided in the README of the respective repositories.

```python
from flask import Flask
# API endpoint modules
from frappyflaskauth import register_endpoints as register_user_endpoints, check_login_state
from frappyflaskdataset import register_endpoints as register_data_endpoints
from frappyflaskcontent import register_endpoints as register_content_endpoints
# storage modules
from frappymongouser import UserStore, UserTokenStore
from frappymongodataset import DataStore
from frappymongocontent import ContentStore

if __name__ == "__main__":
    # create flask app
    app = Flask(__name__)
    mongo_url, mongo_db = "mongodb://localhost:27017", "myDatabaseName"

    # initialise stores
    stores = {
        "user": UserStore(mongo_url=mongo_url, mongo_db=mongo_db, collection_name="users"),
        "user_tokens": UserTokenStore(mongo_url=mongo_url, mongo_db=mongo_db, collection_name="userTokens"),
        "data": DataStore(mongo_url=mongo_url, mongo_db=mongo_db, collection_name="dataSets"),
        "content": ContentStore(mongo_url=mongo_url, mongo_db=mongo_db, collection_name="content"),
    }

    # register endpoints for authentication and user management
    register_user_endpoints(app, stores["user"], stores["user_tokens"])
    # register endpoints for data management and retrieval
    register_data_endpoints(app, stores["data"], {
        "manage_permission": "manage",  # permission required to manage content
        "login_check_function": check_login_state,  # this function ties the authentication module into this one
    })
    # register endpoints for content management
    register_content_endpoints(app, stores["content"], {
        "manage_permission": "manage",
        "login_check_function": check_login_state,
    })
```

_Please also register static endpoints for the frontend (see the `static_api.py` example in the project template)_

### NodeJS Backend

```javascript
import express from "express"
import bodyParser from "body-parser"
import mongodb from "mongodb"
// storage packages
import { UserStore, UserTokenStore } from "@frappy/js-mongo-user-store"
import { DataSetStore } from "@frappy/js-mongo-dataset-store"
import { ContentStore } from "@frappy/js-mongo-content-store"
// backend packages
import { registerEndpoints as registerUserEndpoints, authMiddleware } from "@frappy/node-authentication"
import { registerAdminEndpoints, registerGetEndpoints } from "@frappy/node-datasets"
import { registerAdminEndpoints as registerAdminContent, registerGetEndpoints as registerGetContent } from "@frappy/node-content"

// create and configure main express app
const app = express()
// mount static frontend to express
app.use(express.static(path.join(__dirname, "..", "static")))
// mount parser for application/json content
app.use(bodyParser.json({ limit: "100mb" }))

// connect to database and instantiate database stores
mongodb.MongoClient.connect("mongodb://localhost:27017", { useNewUrlParser: true }).then(client => {
    // initialise store
    const userStore = new UserStore(client, "myDatabaseName", "users")
    const userTokenStore = new UserTokenStore(client, "myDatabaseName", "userTokens")
    const dataSetStore = new DataSetStore(client, "myDatabaseName", "dataSets")
    const contentStore = new ContentStore(client, "myDatabaseName", "content")

    // this is the cache which will contain active auth tokens while the server is running
    const tokenCache = {}

    // register endpoints for authentication and user management
    registerUserEndpoints(app, userStore, userTokenStore, tokenCache)
    // register endpoints for data sets (separate endpoints for management and retrieval)
    registerAdminEndpoints(app, dataSetStore, authMiddleware("manage", tokenCache))  // requires "manage" permission
    registerGetEndpoints(app, dataSetStore, authMiddleware(null, tokenCache))  // requires just to be logged in

    // register endpoints for content
    registerAdminContent(app, contentStore, authMiddleware("manage", tokenCache))
    registerGetContent(app, contentStore, authMiddleware(null, tokenCache))
})

// Start the app
const HTTP_PORT = process.env.PORT || 3000
app.listen(HTTP_PORT, () => {
    console.log(`Listening on port ${HTTP_PORT}`)
})
```

### React Frontend

This is a basic example of an application that is protected with authentication and provides separate routes for the
 management of the types of objects. Please note that you have to **register those routes** with the backend as well, if
 you are using the Python (Flask) backend. If you use NodeJS template, those endpoints are covered by a regular
 expression and no customisation is required.

```javascript
import React from "react"
import { render } from "react-dom"
import { withRouter, Switch, Route } from "react-router"
import { BrowserRouter } from "react-router-dom"

// imports for Frappy modules
import { LoginWrapper, UserManager, PermissionCheck } from "@frappy/react-authentication"
import { DataSetManager } from "@frappy/react-datasets"
import { ContentManager } from "@frappy/react-content"

const RouterApp = withRouter(props => (
    <div>
        <Switch>
            {/* add your other routes here as well, e.g. `/demo` or `/` (home) */}
            <Route path="/" exact component={...} />

            {/* user management route */}
            <Route path="/admin/user" exact component={() =>
                <UserManager currentUser={props.currentUser} permissions={["view", "manage", "admin"]} />
            }/>

            {/* data set management route */}
            <Route path="/admin/data" exact component={() =>
                {/* requires currentUser to have permission 'manage' */}
                <PermissionCheck currentUser={props.currentUser} requiredPermissions="manage" showError>
                    {/* provide data set management UI with data assignments (data set can be assigned to data type)*/}
                    <DataSetManager assignments={{
                        "assignmentKey1": {
                            "label": "Assignment Object #1",
                            "dataTypes": ["type1", "type2", "type3"]
                        },
                        "assignmentKey2": {
                            "label": "Assignment Object #2",
                            "dataTypes": ["type4", "type5"]
                        }
                    }} />
                </PermissionCheck>
            }/>

            {/* content management route */}
            <Route path="/admin/content" exact component={() =>
                <PermissionCheck requiredPermissions="manage" currentUser={props.currentUser}>
                    {/* provides content management interface with 2 references and 3 different content types */}
                    <ContentManager
                        references={["demo1", "demo2"]}
                        contentTypes={{
                            description: {
                                list: false,
                                fields: ["title", "description"],
                            },
                            team: {
                                list: true,
                                fields: ["name", "role"],
                            },
                            papers: {
                                list: true,
                                fields: ["title", "authors", "publication", "year"],
                            },
                        }}
                    />
                </PermissionCheck>
            }/>
        </Switch>
    </div>
))

// this component wraps the main routing component (above) and ensures the user is authenticated
class MainApplication extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            currentUser: null,  // stores currently logged in user
        }
        this.setUser = this.setUser.bind(this)  // handler for post-login
    }

    setUser(user) {
        // log-in successful, update this component's currentUser state
        this.setState({
            currentUser: user,
        })
    }

    render() {
        return (
            <BrowserRouter>
                <LoginWrapper setUser={this.setUser}>
                    {/* only render RouterApp, if the user is logged in */}
                    <RouterApp currentUser={this.state.currentUser} />
                </LoginWrapper>
            </BrowserRouter>
        )
    }
}

// inject into the index.html into the <div id="root"></div>
render(<MainApplication />, document.getElementById("root"))
```
