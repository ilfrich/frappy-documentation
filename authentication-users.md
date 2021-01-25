# Authentication and Users

1. [Authentication](#authentication)
    1. [Frontend](#frontend)
    2. [Backend](#backend)
2. [User Management](#user-management)

Authentication and user management go hand in hand. You need to be able to adjust user permissions and you want to
 manage the local user accounts (create, change password, update permissions, delete).

## Authentication

Authentication is a 2 part process:

- The **frontend** needs to provide a login form, submit credentials to the server, fetch the received authentication
 token after successful login and use that token for subsequent REST requests.
- The **backend** needs to provide login endpoints, generate the auth token and provide authentication checks for all
 endpoints that the developer wants to protect.

### Frontend

The frontend part is pretty simple. The following components related to authentication are provided by the
 [`@frappy/react-authentication`](https://github.com/ilfrich/frappy-react-authentication) package:

Components rendering children if a check is successful:
- `LoginWrapper` to perform some basic "are-you-logged-in" check and renders a login form if necessary.
- `PermissionCheck` which checks whether the currently logged in user has a specific permission.

Action Components:
- `Logout` which has an `onClick` handler that logs you out.

You can see more details in the README.md of that package.

**Additional Backend Calls**

If you develop your own functions to access the backend, you can use the
[`quick-n-dirty-utils`](https://www.npmjs.com/package/quick-n-dirty-utils) package to streamline your `fetch` call:

```javascript
import util from "quick-n-dirty-util"

fetch("/api/url", {
    headers: util.getAuthJsonHeader()
}).then(util.restHandler).then(responseData => {
    console.log("This is already parsed as JSON", responseData)
})
```

### Backend

The **Python** package [`frappyflaskauth`](https://github.com/ilfrich/frappy-flask-authentication) only works with Flask
 and provides:

- the login endpoints with the function `register_endpoints` and
- a function to check the login state `check_login_state`, which also can check permissions and returns the logged in
 user or aborts the Flask request

The **NodeJS** package [`@frappy/node-authentication`](https://github.com/ilfrich/frappy-node-authentication) only works
 with Express and provides:

- the login endpoints with the `registerEndpoints` function and
- an Express middleware `authMiddleware`, which checks whether a user is logged in and can also check permissions.

You can see more details in the README.md of that package.

## User Management

User Management is enabled by default on the backend, if you `register_endpoints` (Python) or `registerEndpoints`
 (NodeJS). See the [Backend](#backend) section above about authentication and the respective package repository's
 README.

For the frontend, the [`@frappy/react-authentication`](https://github.com/ilfrich/frappy-react-authentication) package
 provides the component `UserManager`.

By default the user management will automatically allow to set the `admin` permission and require the `admin` permission
 to manage users. This however can be overwritten in the various authentication modules (backend and frontend).
