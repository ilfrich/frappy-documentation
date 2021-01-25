# Databases

Frappy currently plans to support 2 different databases: **MongoDB** and **MySQL**.
Current state of implementation: MongoDB only.

1. [De-Serialised Object Classes](#de-serialised-object-classes)
2. [MongoDB](#mongodb)
    1. [Base Classes](#base-classes)
    2. [Extending Database Stores](#extending-database-stores)

## De-Serialised Object Classes

Each database store provides a class in the respective language of the store that models the data structure. In Python
 these classes inherit from `AbstractMongoDocument` or `AbstractMysqlDocument` (providing a unique `_id` or `id` of the
 document).

**Python**

Each model class provides a `to_json()` and `from_json(json)` (`@staticmethod`) method that allows serialisation and
 de-serialisation from or into JSON (aka `dict`).

Each database package will also export the object class. However, it can also be retrieved via a store instance. So, if
 you have an instance of an `AbstractMongoStore`, you can get the class via `instance.object_class`:

```python
from frappymongodata import DataStore
store = DataStore(mongo_url="mongodb://localhost:27017", mongo_db="mydb", collection_name="colName")
# this gives you access to the object CLASS
Object_class = store.object_class
# de-serialise an INSTANCE of the class from a dictionary (needs to follow the model structure)
object = Object_class.from_json({ "_id": "abc", "type": "IMAGE", "payload": { ... } })
```

## MongoDB

There are several store packages for NodeJS and Python supporting storage of different objects:

- User and auth token storage
- Data storage
- Content storage

A store represents a single collection of documents within a specific database. You can have multiple collections per
 database for the different types of objects you want to store.

### Base Classes

- Python store classes inherit from `AbstractMongoStore` of the
 [`pbu`](https://github.com/ilfrich/python-basic-utils#abstractmongostore) pip package:
- NodeJS store classes inherit from `AbstractMongoStore` of the `@frappy/js-mongo-store` npm package:

### Extending Database Stores

You can create your own stores based on the [base classes](#base-classes) or extend existing store classes.

**Python example**

```python
from frappymongodata import DataStore

class ProjectContentStore(DataStore):
    # add your own method to
    def get_large_images(min_width, min_height):
        return super().query({
            "payload.dimensions.width": { "$gte": min_width },
            "payload.dimensions.height": { "$gte": min_height },
        })


# create an instance of your new store class (constructor provided by DataStore)
store = ProjectContentStore(mongo_url="mongodb://localhost:27017", mongo_db="mydb", collection_name="colName")

# use your new method
large_images = store.get_large_images(1200, 800)
for image in large_images:
    print("Found a large image:", image.payload.image_path)

# you can still use existing methods of the parent class (DataStore)
from frappymongodata import DataTypes
all_images = store.get_by_type(DataTypes.IMAGE)
```
