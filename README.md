# wwebjs-mssql
A SQL Server (MSSQL) plugin for whatsapp-web.js! 

Use MssqlStore to save your WhatsApp MultiDevice session on a SQL Server Database.

## Quick Links

* [Guide / Getting Started](https://wwebjs.dev/guide/authentication.html)
* [GitHub](https://github.com/paulvl/wwebjs-mysql)
* [npm](https://www.npmjs.com/package/wwebjs-mssql)

## Installation

The module is now available on npm! `npm i wwebjs-mssql`

## Example SQL Server Table statement

```sql
CREATE TABLE [dbo].[wsp_sessions] (
  [id] BIGINT IDENTITY(1,1) NOT NULL,
  [session_name] NVARCHAR(255) NOT NULL,
  [data] VARBINARY(MAX),
  [created_at] DATETIME2 DEFAULT CURRENT_TIMESTAMP,
  [updated_at] DATETIME2 DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT [PK_wsp_sessions] PRIMARY KEY CLUSTERED ([id]),
  CONSTRAINT [UQ_wsp_sessions_session_name] UNIQUE ([session_name])
);

```

## Example usage

```js
const { Client, RemoteAuth } = require('whatsapp-web.js');
const { MssqlStore } = require('wwebjs-mssql');
const sql = require('mssql');

const pool = new sql.ConnectionPool({
    server: process.env.DB_MSSQL_HOST,
    user: process.env.DB_MSSQL_USER,
    password: process.env.DB_MSSQL_PASSWORD,
    database: process.env.DB_MSSQL_DATABASE,
    options: {
        encrypt: true,   // Required for Azure
        trustServerCertificate: true // For local dev/testing
    },
    pool: {
        max: 10,  // max number of concurrent connections
        min: 0,
        idleTimeoutMillis: 30000
    }
});

const tableInfo = {
    table: 'wsp_sessions',
    session_column: 'session_name',
    data_column: 'data',
    updated_at_column: 'updated_at'
}

const store = new MssqlStore({ pool: pool, tableInfo: tableInfo });

const client = new Client({
    authStrategy: new RemoteAuth({
        store: store,
        backupSyncIntervalMs: 300000,
        session: 'your-session-name'
    })
});

client.initialize();

```

## Delete Remote Session

How to force delete a specific remote session on the Database:

```js
await store.delete({session: 'your-session-name'});
```