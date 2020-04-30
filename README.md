# MongoDB to Grafana

This project is based on [simple-json-datasource](http://github.com/grafana/simple-json-datasource) which seems to be discontinued at the moment.

We are creating a backend that implements 4 endpoints that is necessary to implement custom grafana datasource:

- `/` Used for test the connection with the datasource
- `/search` Used by the query tab in panels
- `/query` Return metrics based on input
- `/annontations` Return annotations

Inside the `server` folder there's a express server that exposes these endpoints and make the proxy to the mongodb.

The `src` folder contains some html partials, css and images that will be used to create the configuration page of the custom datasource in the grafana project. The grunt `build` task will use these files to create the final `dist` folder.
The `plugin.json` is mandatory for the plugin, grafana will scan for this file inside the directory. I'm following the schema recommended by the grafana documentation [here](https://grafana.com/docs/grafana/latest/plugins/developing/plugin.json).
The `datasource.js` should contain 4 funcions: 
```javascript
query(options)
testDatasource()
annotationQuery(options)
metricFindQuery(options)
```

The API is expecting values from the query with the following fields: `name`, `value` and `ts`. To accomplish that we are using the Mongo aggregation pipeline.

Todo:
- [] Rewrite the express server
- [] Use gulp instead of grunt to run the tasks
