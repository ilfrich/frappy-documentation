# Data Sets

Data sets are elements that can contain a single type of a data and provide some meta information as well as the payload
of the data set or a reference to the payload (e.g. a file path).

1. [Backend](#backend)
2. [Frontend](#frontend)
3. [Types of Data Sets](#data-set-types)
    1. [Time Series](#time-series)
    2. [Images](#images)
4. [Assignments](#assignments)

## Backend

Simply hook in the endpoints and associate them with the stores as described in the README of the backend repositories:

- [`frappyflaskdatasets`](https://github.com/ilfrich/frappy-flask-datasets) - `register_endpoints`
- [`@frappy/node-datasets`](https://github.com/ilfrich/frappy-node-datasets) - `registerAdminEndpoints` (for data set
 administration) and `registerGetEndpoints` for simple data set retrieval.

The Python package allows to set different permission levels for GET operations and another permission level for
 modifications.

The Javascript package has 2 separate methods due to the fact that you have to provide the required permission to the
 `authMiddleware` _BEFORE_ passing it into the registration function.

## Frontend

For the frontend, there is obviously the `DataSetManager` provided by
 [`@frappy/react-datasets`](https://github.com/ilfrich/frappy-react-datasets), which allows the logged in user to
 perform CRUD operations on data sets.

Additionally, for display purposes, the following components are provided

- `TimeSeriesChart` allows to display a Plotly chart that shows the data set in question, if it is a time series. This
 component requires the data set object itself to be passed in for the component to render.
- `ImageView` will load the full sized image from the API and render it in an image tag. It only requires the image URL,
 which by default is something like `/api/data-sets/<data-set-id>/image` (replace the `data-set-id` with the `_id` of
 the data set).

## Data Set Types

Each data set contains some meta information (label, type, user who created it), assignments and the payload.

The system current supports the following 2 data types:

- Time Series (provided as CSV file)
- Images (provided as single image file)

Future data types - conceptualised - are:

- JSON (generic) - provided as JSON file or directly via copy/paste
- Image sets (zip file full of images)
- Audio files
- Video files

### Time Series

A time series is uploaded as CSV. On upload, the user has the ability to provide custom labels for the various columns
 and also declare the index column (the column containing the date/time information).

Additionally, the user can provide a custom date format using the Python `datetime.strptime` (e.g. `%y-%m-d`) format or
MomentJS's date format (`YYYY-MM-DD`). If the index column provides values as Unix timestamps (in seconds or
milliseconds), you can leave the date format empty.

The data set payload will include:

- `columns` - original list of columns in the CSV file
- `columnMapping` - a JSON object mapping from the original column name to any custom label provided
- `indexColumn` - the name of the original column in the CSV that represents the date/time information
- `dateFormat` - the provided date format
- `data` an array of arrays, the first level representing each row, and inside each column (**PLEASE NOTE**: the data
 is provided as strings! CSV data is not parsed when ingested to numeric values, but remains as string.)

### Images

Images can be uploaded as image and are stored in the `dataFolder` or `data_folder` (passed as option to the backend
 registration functions).

The data set payload will contain:

- `imagePath` - the path to the file in the `dataFolder` / `data_folder` - including the folder and complete file name
- `thumbnail` - a base64 representation of a smaller version of the uploaded image. Thumbnail dimensions are max 100px
 with the aspect ratio retained from the original image.
- `dimensions` - a JSON object with `width` and `height` keys

## Assignments

Data sets can be assigned to a reference and given a specific type. To illustrate the purpose of this take the following
 example.

Imagine you have a demo (let's call it `demo1`) that requires 2 different types of time series: Some sensor data (call
 it `sensorData`) and some additional weather data that supports the algorithm (call it `weatherData`). The assignments
 allow you to upload 2 different CSV files, one representing the `sensorData` for `demo1` and the other one representing
 the `weatherData` for `demo1`.

The assignment model is provided as property to the `DataSetManager` React component:

```javascript
<DataSetManager
    assignments={{
        demo1: {
            label: "Demo 1",
            dataTypes: ["sensorData", "weatherData"],
        },
        demo2: {
            label: "Some other Demo",
            dataTypes: ["sensorData", "someOtherData", "floorPlanImage"],
        },
    }}
/>
```

The `ContentManager` then allows you to select those assignments and assign data types to created data sets.

The types do not restrict the type of data set you want to use and `dataTypes` in 2 different assignment groups can have
 the same name, but remain unrelated.

The assignments allow you to fetch exactly the data you need, depending on the context of your application that you
 build on top of the data set management, that uses the data sets to perform computations etc.
