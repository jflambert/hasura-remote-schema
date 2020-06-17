# hasura-remote-schema
An attempt to demonstrate Hasura's irritating handling of offline remote schemas as invalid metadata. Please refer to [Issue #5117](https://github.com/hasura/graphql-engine/issues/5117) for more details.
Executing `docker-compose up` will trigger the following sequence:

 1. `postgres` starts and creates three databases: `parent`, `child1`, `child2`
 2. `hasura-parent` starts with two remote schemas: `hasura-child1` and `hasura-child2`
 3. `hasura-child1` starts immediately, and isn't a problem for `hasura-parent`
 4. `hasura-child2` sleeps for 121 seconds during the first migration step.
 5. `hasura-parent` crashes and restarts 10 times while waiting for `hasura-child2`
 6. `hasura-child2` eventually starts, and `hasura-parent` finally starts too.

There are two main problems here

 - `hasura-parent` should be able to start immediately despite one of the remote schemas being "offline". It's no different from being able to add remote schemas after initialization.
 - Given the 60 seconds timeout value,`hasura-parent` should only restart 3 or 4 times ([Issue #5126](https://github.com/hasura/graphql-engine/issues/5126)). It restarts 10 times with this test:
```
$ docker inspect --format "NAME: {{.Name}} RESTARTS: {{.RestartCount}}" $(docker ps -aq)
NAME: /hasura-remote-schema_hasura-child2_1 RESTARTS: 0
NAME: /hasura-remote-schema_hasura-child1_1 RESTARTS: 0
NAME: /hasura-remote-schema_hasura-parent_1 RESTARTS: 10
NAME: /hasura-remote-schema_postgres_1 RESTARTS: 0
```
