# Admin Interfaces

This guide outlines a few options in how to integrate the administration interfaces into your React app.
The examples in this guide are minimal examples and only illustrate the embedding of Frappy components, not their
detailed usage.

1. [General Guidelines](#general-guidelines)
    1. [Separate Routes with Permission Check](#separate-routes-with-permission-check)
2. [Using Component States](#using-component-states)
    1. [Using a String State Variable](#using-a-string-state-variable)
    2. [Using Boolean State Variables](#using-boolean-state-variables)

## General Guidelines

While a React app can survive without a router, it is recommended to have separate routes for separate parts of your
 application. One of the main benefits is, that you can share links that directly point to a part of the app, rather
 than just sending the host name of the app and then having to explain how to navigate to a point of interest.

You can display different content on the page depending on the `state` of a React component. This can either be a toggle
 represented by a boolean variable in the component's state (`true` means show a section, `false` means hide it).

Another option to use the state is to have a string variable in the state that states which part of the application will
 be rendered. And a navigation or sub-navigation can then set the state to different string values that point to the
 content.

## Separate Routes

This basic example simply injects all the admin components into a component with router support. No permission check is
 performed in this example, although the API will decline requests, if the user has insufficient permission.

```javascript
import { Switch, Route, withRouter } from "react-router"
import { DataSetManager } from "@frappy/react-datasets"
import { ContentManager } from "@frappy/react-content"
import { UserManager } from "@frappy/react-authentication"

const RouterApp = (withRouter(props) => (
    <div>
        <Switch>
            <Route path="/admin/user" exact component={() => <UserManager currentUser={props.currentUser} />} />
            <Route path="/admin/data" exact component={() => <DataSetManager currentUser={props.currentUser} />} />
            <Route path="/admin/content" exact component={() => <ContentManager currentUser={props.currentUser} />} />
        </Switch>
    </div>
))
```

### Separate Routes with Permission Check

If you want to check for permissions in above example, simply wrap the Frappy components in `<PermissionCheck>`
 components, provided by the `@frappy/react-authentication` package:

```javascript
<Route path="/admin/data" exact component={() => (
    <PermissionCheck currentUser={props.currentUser} requiredPermissions="manage" showError>
        <DataSetManager currentUser={props.currentUser} />
    </PermissionCheck>
)} />
```

In this example, the route requires "manage" permission from the user and will show an error that indicates that the
 user has insufficient permission. It will only render the `DataSetManager` if the user has this permission.

## Using Component States

There are generally 2 different methods to use the component state to render the backend.

- **Option 1**: Use a string variable in the component's state to differentiate which admin part to show
- **Option 2**: Use multiple boolean variable to show an admin part in a section of the component

### Using a String State Variable

The following component renders a minimal sub-navigation that allows the user to render different admin components. The
 `switchSection` method will return specific event handlers that switch the components `adminSection` state to a new
 value. Depending on the value of that `adminSection` different content will then be rendered.

```javascript
import React from "react"
import { DataSetManager } from "@frappy/react-datasets"
import { ContentManager } from "@frappy/react-content"
import { UserManager } from "@frappy/react-authentication"

class AdminApp extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            adminSection: null,
        }
        this.switchSection = this.switchSection.bind(this)
    }

    switchSection(adminSection) {
        return () => {
            this.setState({ adminSection })
        }
    }

    render() {
        return (
            <div>
                <ul>
                    <li onClick={this.switchSection("data")}>Data Set Manager</li>
                    <li onClick={this.switchSection("content")}>Content Manager</li>
                    <li onClick={this.switchSection("user")}>User Manager</li>
                </ul>
                <div>
                    {this.state.adminSection === "data" ? <DataSetManager currentUser={this.props.currentUser} /> : null}
                    {this.state.adminSection === "content" ? <ContentManager currentUser={this.props.currentUser} /> : null}
                    {this.state.adminSection === "user" ? <UserManager currentUser={this.props.currentUser} /> : null}
                </div>
            </div>
        )
    }
}
```

### Using Boolean State Variables

The following component has multiple sections that are by default hidden, but can be made visible if the user toggles a
 section.

```javascript
import React from "react"
import { DataSetManager } from "@frappy/react-datasets"
import { ContentManager } from "@frappy/react-content"
import { UserManager } from "@frappy/react-authentication"

class AdminApp extends React.Component {
    constructor(props) {
        super(props)
        this.state = {
            showData: false,
            showContent: false,
            showUser: false,
        }
        this.toggleData = this.toggleData.bind(this)
        this.toggleContent = this.toggleContent.bind(this)
        this.toggleUser = this.toggleUser.bind(this)
    }

    toggleData() {
        this.setState(oldState => ({
            ...oldState,
            showData: !oldState.showData,  // inverts the value (false -> true or true -> false)
        }))
    }

    toggleContent() {
        this.setState(oldState => ({
            ...oldState,
            showContent: !oldState.showContent,  // inverts the value (false -> true or true -> false)
        }))
    }

    toggleUser() {
        this.setState(oldState => ({
            ...oldState,
            showUser: !oldState.showUser,  // inverts the value (false -> true or true -> false)
        }))
    }

    render() {
        return (
            <div>
                <h5 onClick={this.toggleData}>{this.state.showData ? "Hide" : "Show"} Data Set Manager</h5>
                {this.state.showData ? <DataSetManager currentUser={this.props.currentUser} /> : null}

                <h5 onClick={this.toggleContent}>{this.state.showContent ? "Hide" : "Show"} Content Manager</h5>
                {this.state.showContent ? <ContentManager currentUser={this.props.currentUser} /> : null}

                <h5 onClick={this.toggleUser}>{this.state.showUser ? "Hide" : "Show"} User Manager</h5>
                {this.state.showUser ? <UserManager currentUser={this.props.currentUser} /> : null}
            </div>
        )
    }
}
```

You can combine the 3 methods `toggleData`, `toggleContent` and `toggleUser` into one, by passing the state key
 (`showData`, `showContent`, `showUser`) into the method when hooking in the `onClick` handler, obviously reducing the
 number of handlers to 1. The disadvantage of that approach is that it is harder to trace down when states change (from
 a debugging perspective), because the state key is no longer referenced directly during the update, but referenced
 indirectly by the `onClick` hook. The biggest issues is probably, when you refactor the code and for example rename the
 state keys. IDEs will find it difficult to figure out that the string parameter used in the event hook is the same as
 the key in the state.

This is how the handler method would look like (method of a class):

```javascript
toggle(stateKey) {
    return () => {
        this.setState(oldState => {
            const update = { ...oldState }
            update[stateKey] = !oldState[stateKey]
            return update
        })
    }
}
```

And then the toggle itself looks like this:

```javascript
<h5 onClick={this.toggle("showData")}>...</h5>
```
