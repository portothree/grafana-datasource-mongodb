# MongoDB to Grafana

Visualize data from MongoDB in Grafana.

This project is based on [simple-json-datasource](http://github.com/grafana/simple-json-datasource) and [mongodb-grafana](https://github.com/JamesOsgood/mongodb-grafana) which both seems to be discontinued at the moment.

We are creating a backend that implements 4 endpoints that is necessary to implement custom grafana datasource:

-   `/` Used for test the connection with the datasource
-   `/search` Used by the query tab in panels
-   `/query` Return metrics based on input
-   `/annontations` Return annotations

Inside the `server` folder there's a express server that exposes these endpoints and make the proxy to the mongodb.

The `src` folder contains some html partials, css and images that will be used to create the configuration page of the custom datasource in the grafana project. The gulp `build` task will use these files to create the final `dist` folder.
The `plugin.json` is mandatory for the plugin, grafana will scan for this file inside the directory. I'm following the schema recommended by the grafana documentation [here](https://grafana.com/docs/grafana/latest/plugins/developing/plugin.json).
The `datasource.js` should contain 4 funcions:

```javascript
query(options);
testDatasource();
annotationQuery(options);
metricFindQuery(options);
```

## Prerequisite

-   Grafana 6.x.x
-   Node 10.x

The API is expecting values from the query with the following fields: `name`, `value` and `ts`. To accomplish that we are using the Mongo aggregation pipeline.

## Usage

### Query Editor

-   Only **aggregate** are supported.
-   You can only run one query.
-   `$from` and `$to` are expanded by the plugin as BSON dates based on the range defined in the UI.

### Visualizing time series data

The plugin needs to know which field to use as the time, you can simply project the field with a name alias of "ts". The field data type must be a date.

Example:

```
db.measurement.aggregate([
  {
    "$match": {
     "timeStamp": { "$gte": "$from", "$lte": "$to" }
    }
  }, {
    "$sort": {
      "timeStamp": -1
    }
  }, {
    "$unwind": "$values"
  }, {
    "$match": {
      "values.field": "CO"
    }
  }, {
    "$project": {
      "_id": 0,
      "name": "$box",
      "value": "$values.value",
      "ts": "$timeStamp"
    }
  }
])
```

Todo:

-   [] Rewrite the express server
-   [x] Use gulp instead of grunt to run the tasks
