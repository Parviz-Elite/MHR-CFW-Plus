# Changelog

## Unreleased

### Added

- Added adaptive Google IP failover via the new `google_ips` and
  `google_ip_fail_cooldown` config options.
- Added failover support to both the HTTP/1.1 Apps Script connection pool and
  the HTTP/2 multiplexed transport.
- Added a local `/status` dashboard and `/status.json` API that report active
  Google IP state, per-IP success/failure counts, cooldowns, H2 status, Apps
  Script health, cache stats, top relay hosts, and a built-in troubleshooting
  FAQ.
- Added a local home page at `/` and a config dashboard at `/dashboard` for
  viewing and safely editing `config.json` from the browser.
- Added dashboard control actions for clearing cache, clearing route cooldowns,
  clearing script blacklists, resetting runtime stats, and reconnecting H2.
- Expanded the dashboard troubleshooting section with common setup, CA, relay,
  Apps Script quota, SOCKS5/Telegram, YouTube, CAPTCHA, H2, and dashboard access
  fixes.
- Added Apps Script health scoring so script latency/failure history is visible
  and fan-out relay can prefer healthier fallback deployments.

### Changed

- Updated `config.example.json`, the setup wizard, and both English and Persian
  READMEs to document Google IP failover, the dashboard, and the status API.
