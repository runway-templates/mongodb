# mongodb

[MongoDB](https://www.mongodb.com) template for Runway. Document database
with authentication and persistence on a volume.

Single instance: no replica set, so no failover and no multi-document
transactions.

## Deploy

WIP

## Inputs

| Input | Default | Notes |
| --- | --- | --- |
| `image_tag` | `8.0` | `8.0`/`8.3` supported to Oct 2029, `7.0` to Aug 2027 |
| `database_name` | `app` | Created on first write, not at startup |
| `username` | `app` | Root user in the `admin` database |

## Connecting

Not routed — reachable only from inside your organization's network, at:

```
<app>.<app>.svc.cluster.local:27017
```

`<app>` is your app name.

The credentials are in the app config:

```sh
runway app config ls --app <app>
```

Authenticate against `admin`, which is where the user is created:

```
mongodb://<username>:<MONGO_INITDB_ROOT_PASSWORD>@<app>.<app>.svc.cluster.local:27017/<database_name>?authSource=admin
```

## Notes

- **The password only applies on the first start**, when the data directory
  is empty. Changing `MONGO_INITDB_ROOT_PASSWORD` later does not change it in
  MongoDB — it just leaves your clients with a value that no longer works.
  Rotate over a connection instead:

  ```js
  db.getSiblingDB("admin").changeUserPassword("app", "<new>")
  ```

  Then update the config to match.

- **The template sets no `command`.** The image entrypoint is what reads the
  `MONGO_INITDB_*` variables and creates the user before starting `mongod
  --auth`. Overriding the command skips that and leaves a database nobody can
  log into.

- **Major upgrades go one release at a time** and need
  `featureCompatibilityVersion` set before each step. Do not jump `image_tag`
  from 7.0 to 8.3.

## License

MongoDB is SSPL-1.0 — read it before using this commercially. This template
is maintained by Runway.
