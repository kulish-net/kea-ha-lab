# kea-ha-lab

A reproducible lab for **ISC Kea DHCPv4 in hot-standby high availability** with a
shared PostgreSQL lease backend — everything runs in containers, no hardware and
no cloud account required.

Built to answer a question that comes up whenever DHCP stops being "one box in a
closet": *what actually happens to client leases when the primary server dies?*

## Topology

```
        172.28.0.0/24  (docker bridge "lab")

  ┌────────────────┐        ┌──────────────────┐
  │ kea-primary    │◄──HA──►│ kea-secondary    │
  │ 172.28.0.2     │        │ 172.28.0.3       │
  └───────┬────────┘        └────────┬─────────┘
          │        leases (SQL)      │
          └────────────┬─────────────┘
                       ▼
              ┌─────────────────┐      ┌──────────────┐
              │ postgres        │      │ client       │
              │ 172.28.0.10     │      │ 172.28.0.50  │
              └─────────────────┘      └──────────────┘
```

## Quick start

```bash
docker compose up -d
docker compose logs -f kea-primary
```

Request a lease from the client container, then kill the primary and watch the
standby take over:

```bash
docker compose exec client sh -c 'apt-get update && apt-get install -y isc-dhcp-client && dhclient -v eth0'
docker compose stop kea-primary
docker compose exec client dhclient -v -r eth0 && docker compose exec client dhclient -v eth0
```

## What this lab demonstrates

- `hot-standby` HA mode and how it differs from `load-balancing`
- failover behaviour driven by `max-unacked-clients` / `max-response-delay`
- why a shared lease database is **not** by itself high availability
- lease continuity across a failover from the client's point of view

## Status

Skeleton — configs are written but not yet validated end to end. Known open
points, to be resolved and documented as the lab is built:

- [ ] Confirm the current ISC Kea image name and tag
- [ ] HA peers talk over HTTP on `:8000`; this needs either `kea-ctrl-agent`
      alongside each server, or Kea's dedicated HA listener with
      multi-threading enabled — decide which and document why
- [ ] PostgreSQL schema must be created (`kea-admin db-init pgsql`) before first
      start; wire this into `db/init/`
- [ ] Verify `hooks-libraries` paths inside the image

## License

MIT
