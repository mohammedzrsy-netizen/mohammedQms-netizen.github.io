# Meeza QMS — Milestone 3 (Security + Hybrid Sync + Printer Adapter)

This increment extends the Milestone 1 starter with:
- Queue lifecycle endpoints: call-next, recall, hold, resume, transfer, cancel, no-show, complete.
- A bilingual Arabic/English browser prototype for teller, kiosk, public display, and basic administration.
- A minimal site event outbox schema and sync design notes for hybrid deployments.
- Local run instructions and release limitations.

## New in Milestone 3
- Staff authentication is required for ticket creation and queue listing.
- Organization/branch scope checks added to those routes.
- Site-initiated sync publisher (`python -m app.sync_worker`) with retries and event IDs.
- Central event receiver with bearer-token validation and idempotent receipts.
- Printer adapter contract: browser print implementation and explicitly unimplemented ESC/POS adapter until printer model is confirmed.
- Third Alembic migration for central sync receipts.
- Initial tests for ticket sequence scoping and printer adapter behavior.

## Start
1. Unzip the project.
2. Copy `.env.example` to `.env`, replace all passwords/secrets.
3. Run `docker compose up --build -d`
4. Run `docker compose exec api alembic upgrade head`
5. Open `http://localhost:8000/ui/` for the prototype.

The UI is a functional API-connected prototype, not a production-ready polished product. It requires initial organization, branch, service, counter, and admin records to be provisioned securely. Do not expose it publicly before completing the security checklist.

## Migration note
This package includes a second Alembic migration. Back up the database before upgrading. Test upgrade and rollback in a staging copy first.

## Production blockers
- All business endpoints must be protected with complete authentication and organization/branch authorization.
- The demo UI does not implement OIDC/Entra ID, MFA, secure user provisioning, or full audit UI.
- Site-to-central synchronization has a durable outbox schema and design, but not a complete sync worker or conflict-resolution engine.
- Kiosk print support is browser print-based; certified printer adapters, fiscal/thermal printer integration, and device-specific testing remain required.
- Public display refresh currently uses polling; production should use WebSocket/SSE and resilient reconnection.
- Branch-local business date and daylight-saving policy must be implemented before relying on ticket sequence reset behavior.

## Run tests
After installing requirements, run `pytest -q`. For production acceptance, add PostgreSQL concurrency/integration tests and execute them against the target database version.

## Run site sync publisher
Set `CENTRAL_SYNC_URL` and `CENTRAL_SYNC_TOKEN` in `.env`, then run `python -m app.sync_worker` in a separate process/container. The central receiver currently stores event receipts and payloads; it does not yet apply every event to central business projections.
