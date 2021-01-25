# Data Structures

This document outlines the various different data structures of content and data sets.

The general data format of objects (data sets, content objects, users) is JSON. This format is native to JavaScript
 (frontend) and NodeJS (backend). The Python backend provides classes of these objects, which have attributes with the
 same or a similar name as the JSON attribute. Since JSON uses `camelCase` and Python uses `snake_case`. Examples:

- JSON `object.payload` -> Python `object.payload`
- JSON `object.indexColumn` -> Python `object.index_column`
- JSON `object._id` -> Python `object.id` (this is an exception to the rule and is more related to MongoDB's internal
 storage)

If you prefer a Python `dict` to represent the object, then you can use the **`to_json()`** method that every object
 provides. It returns a dictionary, with the JSON naming convention (`camelCase`).

Sub-fields that are described in this document as "JSON object / dictionary" will only use `camelCase` keys!

## Content

```json
{
    "_id": "5ea0165beaf1d2b82b6dcd93",
    "content": {
        "title": "Frappy Demo",
        "description": "This is the description to display for the demo."
    },
    "references": ["frappyDemo"],
    "contentType": "description",
    "label": "Main Description"
}
```

**Attributes**

References to `ContentManager` indicate that the values or structure is defined by the embedding of the `ContentManager`
 into the front-end code. It's there, that the developer defines what types of content with what structure is available.

- `_id` (`id`): the internal ID used to uniquely identify this object
- `content`: this is the payload of the content object, containing the actual content provided by the user in the
 administration panel. There are 2 options for this structure, depending on whether the content object represents a list
 or a flat content object:
   - ***Flat Object***: a JSON object / dictionary containing keys for each field defined by the content definition
    (`ContentManager`).
   - ***List***: an array containing JSON objects / dictionaries for each row provided by the user with the
    corresponding columns (`ContentManager`).
- `references`: an array of strings that define associations with pre-defined strings provided in the front-end
 (`ContentManager`).
- `contentType` (`content_type`): an identifier for the type of content as defined in the front-end (`ContentManager`)
- `label`: the label provided by the user when the content object was created.

## Data Sets

There are 3 different types of data sets: `TIME_SERIES` (CSV), `IMAGE` (.jpg, .png, ...) and `JSON` (generic data).

**References**

Each data set can have `assignments`, which are a specification of ***references*** and ***dataTypes***:

```json
{
    "_id": "5ea0165beaf1d2b82b6dcd93",
    "assignments": {
        "frappyDemo": ["history_data"]
    },
    ...
}
```

This mechanism allows the developer to specify groups of data sets (e.g. related to a specific problem) with different
 types (e.g. different type of input data). The purpose of this is to allow grouping data sets to a problem, but still
 being able to differentiate data sets within that problem.

Example: a ***reference*** could be `demoWeatherForecast` (referring to a problem, forecasting the weather) and then you
 could have a ***dataType*** `history_data`, which gives historic data and `current_data`, which gives the current
 situation. That allows the user on upload of these data sets to assign specific roles within a problem and the
 developer for the backend later to pick the right data set according to its role within the problem for the inputs of
 the algorithm.

 Each data set has a couple of base fields and then a **`payload`**, which contains content specific to the type of data
 set.

**Base Attributes**

- `_id` (`id`): the internal ID used to uniquely identify this object.
- `type`: a string representing the type of data set. One of: `TIME_SERIES`, `IMAGE`, `JSON`.
- `label`: the name of the data set provided by the user who uploaded it.
- `userId` (`user_id`): a string representing the user who uploaded the data set (email or username).
- `assignments`: a JSON object / dictionary containing keys for the main `references` (see description above) and each
 reference key contains an array of values representing the `dataTypes` within that `reference` that has been assigned
 to them on upload.
- `payload`: a JSON object / dictionary representing the payload specific to the type of data set (see below)

### Image Data Set Payload

This is the structure of the `payload` value of an `IMAGE` data set:

```json
{
    "dimensions": {
        "width": 800,
        "height": 600
    },
    "imagePath": {
        "_data/5ea014cbeaf1d2b82b6dcd92.png"
    },
    "thumbnail": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAJYAAABwCAYAAAD8HIl9AAAg..."
}
```

**Attributes**

- `dimensions`: simply, the width and height of the original image
- `imagePath` (`image_path`): the local (relative) path on the backend, where to find the original image in the file
 system
- `thumbnail`: the thumbnail of the original image in base64 format (can be directly included into an `<img>` tag)

### Time Series Data Set Payload

This is the structure of the `payload` value of a `TIME_SERIES` data set:

```json
{
    "columns": ["timestamp", "power", "energy"],
    "columnMapping": {
        "power": "Generated Power",
        "energy": "Consumed Energy"
    },
    "indexColumn": "timestamp",
    "dateFormat": "%y-%m-%d %H:%M",
    "data": [
        ["2020-03-12 14:25", 588.8, 124.3],
        ["2020-03-12 14:30", 624.2, 121.2],
        ["2020-03-12 14:35", 593.3, 124.9],
        ...
    ]
}
```

**Attributes**

- `columns`: lists the original columns of the uploaded CSV in order
- `columnMapping` (`column_mapping`): a JSON object / dictionary that can contain _optional_ labels for the `columns`,
 where the key maps to the strings in the `columns` list and the value is the label provided by the user on upload.
- `indexColumn` (`index_column`): this column is very important for the `TimeSeriesChart` and indicates, which column
 contains the x-axis values (time).
- `dateFormat` (`date_format`): this is optional and only needs to be provided, if the index column contains dates
 represented as strings. The format can either be Python `datetime.strptime` format or JavaScript MomentJS format.
 ***Warning***: the 2 formats are not 100% compatible. Certain fields cannot be defined in one language or the other.
 The data set modules will try to convert the provided format to their respective format needed, but there are certain
 formats that cannot be converted (especially when dealing with time zone formats - e.g. "+10:00").
- `data` an array of arrays representing the CSV data (without the columns). The order of the values in the inner arrays
 maps to the order of the `columns`.
