# Changelog

## Unreleased

### Added

- Added adaptive Google IP failover via the new `google_ips` and
  `google_ip_fail_cooldown` config options.
- Added failover support to both the HTTP/1.1 Apps Script connection pool and
  the HTTP/2 multiplexed transport.
- Added a local `/status` endpoint that reports active Google IP state,
  per-IP success/failure counts, cooldowns, H2 status, script blacklists,
  cache stats, and top relay hosts.

### Changed

- Updated `config.example.json`, the setup wizard, and both English and Persian
  READMEs to document Google IP failover and the status endpoint.
