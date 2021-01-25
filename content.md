# Content Management

1. [Backend](#backend)
2. [Frontend](#frontend)
    1. [Content Types](#content-types)

The content management works on the basic principle of having the content "model" to be defined in the front-end and the
 backend components are just "dumb" receivers and storage that do not require much configuration.

## Backend

Simply hook in the endpoints and associate them with the stores as described in the README of the backend repositories:

- [`frappyflaskcontent`](https://github.com/ilfrich/frappy-flask-content) - `register_endpoints`
- [`@frappy/node-content`](https://github.com/ilfrich/frappy-node-content) - `registerAdminEndpoints` (for content
 administration) and `registerGetEndpoints` for simple content retrieval.

The Python package allows to set different permission levels for GET operations and another permission level for
 modifications.

The Javascript package has 2 separate methods due to the fact that you have to provide the required permission to the
 `authMiddleware` _BEFORE_ passing it into the registration function.

## Frontend

The `ContentManager` React component of [`@frappy/react-content`](https://github.com/ilfrich/frappy-react-content)
 allows to list all existing content, create new content, update existing content and delete content (CRUD).

The component has 2 important parameters:

- `references` - is an array of IDs that you want to use for association. They are used to provide check-boxes for each
 ID and allows to associate a content element to one or multiple of those references. This is to model parent-children
 relationships without too many constraints.
- `contentTypes` - is a more complex model of all available content types

### Content Types

Content types are provided as JSON object. Take the following example and then we'll explain what each of those elements
means:

```json
{
    "description": {
        "list": false,
        "fields": ["title", "description"]
    },
    "team": {
        "list": true,
        "fields": ["name", "role"]
    }
}
```

- **"description"** and **"team"** are the 2 different content types
    - a **"description"** is not a list item, but just a single item and has 2 fields "title" and "description" which
     you can provide text content for in the `ContentManager`.
    - a **"team"** content entry is a listing and can contain `0..n` items with fields "name" and "role", both of them
     represented as text fields. The `ContentManager` allows you to add new lines to a single content item and remove 
     lines.

Example content for these 2 types are:

**`description`** Example

```json
{
    "title": "My custom title",
    "description": "This is the text provided in the text input"
}
```

**`team`** Example

```json
[
    {
        "name": "Peter Ilfrich",
        "role": "Software Engineer"
    },
    {
        "name": "Justin Sane",
        "role": "Lunatic"
    }
]
```
