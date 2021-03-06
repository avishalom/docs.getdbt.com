---
title: "Snowflake"
id: "profile-snowflake"
---

## Authentication Methods

### User / Password authentication

Snowflake can be configured using basic user/password authentication as shown below.

<File name='~/.dbt/profiles.yml'>

```yaml
my-snowflake-db:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: [account id]

      # User/password auth
      user: [username]
      password: [password]

      role: [user role]
      database: [database name]
      warehouse: [warehouse name]
      schema: [dbt schema]
      threads: [1 or more]
      client_session_keep_alive: False

```

</File>

### Key Pair Authentication

To use key pair authentication, omit a `password` and instead provide a `private_key_path` and, optionally, a `private_key_passphrase` in your target. **Note:** Versions of dbt before 0.16.0 required that private keys were encrypted and a `private_key_passphrase` was provided. This behavior was changed in dbt v0.16.0.

<File name='~/.dbt/profiles.yml'>

```yaml
my-snowflake-db:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: [account id]
      user: [username]
      role: [user role]

      # Keypair config
      private_key_path: [path/to/private.key]
      private_key_passphrase: [passphrase for the private key, if key is encrypted]

      database: [database name]
      warehouse: [warehouse name]
      schema: [dbt schema]
      threads: [1 or more]
      client_session_keep_alive: False
```

</File>

### SSO Authentication

To use SSO authentication for Snowflake, omit a `password` and instead supply an `authenticator` config to your target. `authenticator` can be one of 'externalbrowser' or a valid Okta URL.

**Note**: By default, every connection that dbt opens will require you to re-authenticate in a browser. Contact your Snowflake support rep and inquire about turning on the "id token cache" for your account as described [here](https://github.com/snowflakedb/snowflake-connector-python/issues/140#issuecomment-447028785).

<File name='~/.dbt/profiles.yml'>

```yaml
my-snowflake-db:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: [account id]
      user: [username]
      role: [user role]

      # SSO config
      authenticator: externalbrowser

      database: [database name]
      warehouse: [warehouse name]
      schema: [dbt schema]
      threads: [between 1 and 8]
      client_session_keep_alive: False
```

</File>


## Configurations

The "base" configs for Snowflake targets are shown below. Note that you should also specify auth-related configs specific to the authentication method you are using as described above.

### All configurations

| Config | Required? | Description |
| ------ | --------- | ----------- |
| account | Yes | The account to connect to. This will be something like `abc123` or `abc123.us-east-1` for your particular account |
| user | Yes | The user to log in as |
| database | Yes | The database that dbt should create models in |
| warehouse | Yes | The warehouse to use when building models |
| schema | Yes | The schema to build models into by default. Can be overridden with [custom schemas](using-custom-schemas) |
| role | No (but recommended) | The role to assume when running queries as the specified user. |
| client_session_keep_alive | No | If provided, issue a periodic `select` statement to keep the connection open when particularly long-running queries are executing (&gt; 4 hours). Default: False (see note below) |
| threads | No | The number of concurrent models dbt should build. Set this to a higher number if using a bigger warehouse. Default=1 |

### client_session_keep_alive

The `client_session_keep_alive` feature is intended to keep Snowflake sessions alive beyond the typical 4 hour timeout limit. The snowflake-connector-python implementation of this feature can prevent processes that use it (read: dbt) from exiting in specific scenarios. If you encounter this in your deployment of dbt, please let us know in [the GitHub issue](https://github.com/fishtown-analytics/dbt/issues/1271), and work around it by disabling the keepalive.
