# Jack-2 Docker deployment

Information Peacekeeper 1.6.0-beta.1 was deployed on Jack-2 on 21 November 2026.
The source is this repository. Docker Engine and Compose run on Ubuntu 26.04.1;
Jack-2 has no RKE2 role.

```sh
cd /opt/information-peacekeeper
docker compose -p information-peacekeeper \
  -f docker-compose.yml -f deploy/jack2.compose.yml up -d --build
```

The overlay builds `njoy/information-peacekeeper:jack2-20261121`, publishes the
app only on `127.0.0.1:9292`, and disables public self-registration. Docker's
named volume `information-peacekeeper_xorcism-data` persists the SQLite files.
Do not use `docker compose down -v` unless the data is intentionally being
discarded.

Tailscale Serve proxies private HTTPS on
`https://jack2-mgmt.tail66b127.ts.net/login` to the loopback port. It is not a
Funnel endpoint. Verify with a browser-like user agent because the application
blocks the `curl` user agent on HTML pages:

```sh
tailscale serve status
docker ps --filter name=information-peacekeeper
curl -A 'Mozilla/5.0' -fsS http://127.0.0.1:9292/login >/dev/null
```

The first-start admin account is `admin@xorcism.local`. Its temporary password
was reset after a health-check rebuild and stored in the operator's requested
local file outside Git. The first login must change it. Do not log or commit
passwords. No external inference provider is configured by this Compose stack.

The initial Docker build's production audit found five advisories. The pinned
Express patch and refreshed lockfile brought `npm audit --omit=dev` to zero
reported advisories on deployment day. Recheck at each update. A fresh-database
risk-score startup error was corrected by joining incidents to their asset links
instead of querying nonexistent columns on the link table. The subsequent boot
reported normal event-driven risk-score mode with no startup query error.
Bob verified HTTPS login with the stored temporary password and confirmed
`mustChangePassword=true`. A registration POST returned HTTP 403.
