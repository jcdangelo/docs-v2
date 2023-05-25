---
title: Install the InfluxDB v2 JavaScript client library
seotitle: Install the InfluxDB Node.js JavaScript client library
description: >
  Install the Node.js JavaScript client library to integrate with the InfluxDB v2 API.
menu:
  influxdb_cloud_serverless:
    name: Install
    parent: Node.js
influxdb/cloud-serverless/tags: [client libraries, JavaScript]
weight: 100
aliases:
  - /influxdb/cloud-serverless/reference/api/client-libraries/nodejs/install
---


## Install Node.js

1. Install [Node.js](https://nodejs.org/en/download/package-manager/).

2. Ensure that InfluxDB is running and you can connect to it.
   For information about what URL to use to connect to InfluxDB Cloud, see [InfluxDB URLs](/influxdb/cloud-serverless/reference/urls/).

3. In your terminal, create a directory for your Node.js project and change to it.

   ```sh
   mkdir influx-node-app && cd $_
   ```

4. Enter the following command to generate an npm package for your project. 
   The `npm` package manager is included with Node.js.

   ```sh
   npm init -y
   ```

## Install TypeScript

Many of the client library examples use [TypeScript](https://www.typescriptlang.org/).
Follow these steps to initialize the TypeScript project:

1. Install TypeScript and type definitions for Node.js.

   ```sh
   npm i -g typescript && npm i --save-dev @types/node
   ```
2. Create a TypeScript configuration with default values.

   ```sh
   tsc --init
   ```
3. Run the TypeScript compiler. To recompile your code automatically as you make changes, pass the `watch` flag to the compiler.

   ```sh
   tsc -w -p
   ```

## Install dependencies

Use the `@influxdata/influxdb-client` JavaScript client library to write and query data in InfluxDB Cloud Serverless.

Open a new terminal window and install the `@influxdata/influxdb-client` package for querying and writing data:

   ```sh
   npm install --save @influxdata/influxdb-client
   ```

The `@influxdata/influxdb-client-apis` client library package won't work with InfluxDB v3.
It only works with InfluxDB v2 management APIs.

## Configure credentials

The client examples include an [`env`](https://github.com/influxdata/influxdb-client-js/blob/master/examples/env.js) module for accessing your InfluxDB properties from environment variables or from `env.js`.
The examples use these properties to interact with the InfluxDB API.

Set environment variables or update `env.js` with your InfluxDB [bucket](/influxdb/cloud-serverless/organizations/buckets/), [organization](/influxdb/cloud-serverless/organizations/), [token](/influxdb/cloud-serverless/security/tokens/), and [url](/influxdb/cloud-serverless/reference/urls/).

   ```sh
   export INFLUX_URL=https://cloud2.influxdata.com
   export INFLUX_TOKEN=API_TOKEN
   export INFLUX_ORG=ORG_ID
   export INFLUX_BUCKET=BUCKET_NAME
   ```

   Replace the following:
   
   - *`API_TOKEN`*: InfluxDB API token with _write_ permission to the bucket.
   - *`ORG_ID`*: InfluxDB organization ID
   - *`BUCKET_NAME`*: the name of the InfluxDB Cloud Serverless bucket to write to

## Next steps

Once you've installed the client library and configured credentials, you're ready to [write data](/influxdb/cloud-serverless/api-guide/client-libraries/nodejs/write/).

{{< page-nav next="/influxdb/cloud-serverless/reference/client-libraries/v2/javascript/nodejs/write/" keepTab=true >}}