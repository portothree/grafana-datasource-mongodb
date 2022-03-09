# MongoDB to Grafana

Visualize data from MongoDB in Grafana.

This project is based on [simple-json-datasource](http://github.com/grafana/simple-json-datasource) and [mongodb-grafana](https://github.com/JamesOsgood/mongodb-grafana) which both seems to be discontinued at the moment.

We are creating a backend that implements 4 endpoints that is necessary to implement custom grafana datasource:

- `/` Used for test the connection with the datasource
- `/search` Used by the query tab in panels
- `/query` Return metrics based on input
- `/annontations` Return annotations

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

The API is expecting values from the query with the following fields: `name`, `value` and `ts`. To accomplish that we are using the Mongo aggregation pipeline.

## Prerequisites

- Grafana 6.x.x
- Node 10.x

## Installation

1. Run the server with `npm run server` or `node server/mongo-proxy.js`, its defaults to port `3333`.
2. Clone the project in the `/usr/local/var/lib/grafana/plugins` folder
3. Build the plugin front end with `npm run build` or `npx gulp`
4. Make sure the `dist` folder was created
5. Set the `GF_PLUGIN_ALLOW_LOADING_UNSIGNED_PLUGINS` env var to `grafana-datasource-mongodb` or add `allow_loading_unsigned_plugins = grafana-datasource-mongodb` to your `grafana.ini` file inside the `[plugins]` section
6. Restart the Grafana server. If installed via Homebrew, this can be done with `brew services restart grafana`

## Usage

### Query Editor

- Only **aggregate** are supported.
- You can only run one query.
- `$from` and `$to` are expanded by the plugin as BSON dates based on the range defined in the UI.

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

- [] Rewrite the express server
- [x] Use gulp instead of grunt to run the tasks
