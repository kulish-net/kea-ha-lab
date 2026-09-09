# Lease database schema

Kea does not create its own schema. Before the first start the PostgreSQL
database must be initialised with the Kea DHCP schema, normally via:

```bash
kea-admin db-init pgsql -u kea -p kea -n kea -h 172.28.0.10
```

**Open point for this lab:** decide between

1. running `kea-admin` once from the Kea container as a one-shot init service, or
2. mounting Kea's `dhcpdb_create.pgsql` into this directory so that the
   `postgres` image applies it automatically on first boot.

Option 2 is simpler to reproduce but pins the schema version to whatever ships
with the image used. Whichever is chosen, document the reasoning — this is
exactly the kind of detail the vendor docs gloss over.
