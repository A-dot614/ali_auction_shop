# DevOps and Deployment Plan

**Confirmed tooling:** GitHub, GitHub Actions, Docker, PostgreSQL 16, Redis queues, Laravel 13 (PHP), React.js (TypeScript), Flutter.
**Proposed or to confirm:** AWS Canada `ca-central-1` hosting, Datadog or Laravel-native monitoring, Kubernetes (deferred).

## 1. Environments

| Environment | Purpose | Data |
|---|---|---|
| Local | Developer machines, started with Docker Compose | Synthetic seed data |
| Development (shared) | Integration of merged work | Synthetic data |
| Staging | Pre-production, end-to-end tests, payment sandbox | Anonymised or synthetic only |
| Production | Live marketplace | Real data |

Each environment has its own PostgreSQL 16 database, Redis instance, object storage bucket, `APP_KEY`, mail configuration and payment-provider account (Stripe test mode in non-production). Production personal data is never copied into lower environments unless properly anonymised and approved.

## 2. Repository and branching (GitHub)

- Monorepo recommended for the MVP (decision in document 17):
  ```text
  aliauction/
    api/          Laravel 13 application
    web/          React + TypeScript application
    mobile/       Flutter application (Phase 2)
    contracts/    OpenAPI document and generation scripts
    infra/        Docker, Compose files, deployment templates
    .github/      Workflows, issue and pull request templates, CODEOWNERS
  ```
- `main` is protected: pull request required, at least one approving review, status checks required, no force pushes.
- Short-lived feature branches. Squash merge. Conventional commit messages.
- `CODEOWNERS` assigns reviewers for `api/`, `web/`, `mobile/`, `infra/` and any payment or auction code.
- Secret scanning and push protection enabled on the repository.
- Dependabot enabled for Composer, npm, pub and Docker base images.

## 3. Containerisation (Docker)

Separate images or process targets:

| Image | Base (pinned) | Contents |
|---|---|---|
| `api` | `php:8.4-fpm` (or the version required by Laravel 13), pinned by digest | Laravel application, PHP extensions for PostgreSQL and Redis |
| `web` | `nginx` (pinned) | Nginx configuration and the built React static files |
| `worker` | Same image as `api` | `php artisan horizon` |
| `scheduler` | Same image as `api` | `php artisan schedule:work` (single replica) |
| `reverb` | Same image as `api` | `php artisan reverb:start` |
| `node-build` | `node:22-alpine` (pinned), build stage only | `npm ci` and `npm run build` for React |

Rules:
- Multi-stage builds. Development tools and Composer `--dev` packages are excluded from production images.
- Run as a non-root user. Read-only root filesystem where possible. Writable paths are explicit volumes.
- Images are built with `composer install --no-dev --optimize-autoloader` and `npm ci`.
- Lockfiles (`composer.lock`, `package-lock.json`, `pubspec.lock`) are committed and used in every build.
- Health checks: `api` exposes `GET /api/health`. Workers report heartbeat through Horizon.
- Configuration comes from environment variables. No `.env` file is baked into images.
- PostgreSQL 16 and Redis versions are pinned in Compose and in the managed service configuration.

Local development uses Docker Compose with these services: `api`, `web` (Vite dev server), `worker`, `scheduler`, `reverb`, `postgres:16`, `redis`, and a local mail catcher (Mailpit).

## 4. CI/CD pipeline (GitHub Actions)

Triggered on pull requests and on pushes to `main`.

Pull request checks:
1. Install dependencies from lockfiles (`composer install`, `npm ci`).
2. Format and lint: Laravel Pint, ESLint, Prettier check.
3. Static analysis: Larastan (PHPStan) and `tsc --noEmit`.
4. OpenAPI check: regenerate the document and fail if it differs from the committed file.
5. Unit and feature tests with Pest and Vitest, against a PostgreSQL 16 and Redis service container.
6. Dependency audit: `composer audit`, `npm audit --omit=dev`.
7. Secret scan and static security analysis (CodeQL where supported).
8. Build Docker images.
9. Scan images for known vulnerabilities (Trivy or equivalent).

Merge to `main`:
10. Push immutable images to GitHub Container Registry (GHCR), tagged with the commit SHA.
11. Deploy to staging.
12. Run database migrations as a separate release step.
13. Run smoke tests, then Playwright end-to-end tests and the Flutter integration suite against staging.
14. Production deployment waits for a manual approval from the release manager (GitHub Environments with required reviewers).
15. Deploy with health checks. Keep the previous image for rollback.

Flutter release pipeline (Phase 2): build signed Android app bundles and iOS archives in GitHub Actions. Upload to Google Play and TestFlight through store credentials stored as encrypted GitHub secrets.

## 5. Database migrations

- Migrations are Laravel migrations, versioned in `api/database/migrations`.
- Use expand and contract changes: add new columns as nullable, backfill in a queued job, switch reads, then remove old columns in a later release.
- Avoid long-running table locks during peak bidding. Use `CREATE INDEX CONCURRENTLY` in PostgreSQL for large tables, configured with `$withinTransaction = false` on that migration.
- Test every migration on a copy of production-like volume before release.
- Back up before any risky production migration.
- Prefer forward fixes for irreversible data changes.
- Enum-like fields use `string` columns with CHECK constraints rather than native PostgreSQL enum types, so that values can be added with a simple migration.

## 6. AWS deployment baseline (to confirm)

Proposed components:
- VPC with private subnets for database and Redis.
- Application Load Balancer with HTTPS (ACM certificate).
- ECS on Fargate for `web`, `api`, `worker`, `scheduler` and `reverb`. The scheduler runs as exactly one task.
- RDS for PostgreSQL 16 with Multi-AZ in production.
- ElastiCache for Redis.
- S3 buckets for media and backups, with versioning and encryption.
- CloudFront in front of public media and the React build.
- Secrets Manager and KMS.
- CloudWatch logs and alarms, or Datadog (decision).
- AWS Backup for scheduled, encrypted backups.

Confirm service availability and region for each component before the final topology is chosen.

Reverb requires WebSocket support on the load balancer and sticky-free routing, with Redis pub/sub enabled for multiple instances.

## 7. Kubernetes (deferred)

Kubernetes is not part of the MVP. Consider it only when the team has the operational capacity and the workload requires it. If adopted, it must include resource requests and limits, probes, pod disruption budgets, network policies, secret integration, TLS ingress, upgrade planning and runbooks. PostgreSQL and Redis stay on managed services.

## 8. Queue and scheduler operations

- Horizon runs as the queue supervisor. Queues are separated by priority: `critical` (auction close, payments), `default` (notifications), `low` (search indexing, images, analytics).
- Horizon metrics: throughput, runtime, failed jobs, queue wait time. Alerts when wait time exceeds the threshold agreed in document 14.
- Failed jobs are retried with backoff and then stored in `failed_jobs`. Operators can retry or discard them from Horizon, and each action is audited.
- The scheduler container runs one replica. A second replica could double-run commands; the auction-close command is idempotent in any case.
- Use `php artisan queue:work --tries` and `--timeout` values that match each job type.

## 9. Observability

Options (decision required): Datadog, or Laravel Pulse plus Sentry and CloudWatch.

Monitor:
- API request rate, latency (p50, p95, p99) and error rate per endpoint.
- Bid acceptance, rejection and contention rates.
- Auction close job delay and failures.
- Payment webhook lag and reconciliation mismatches.
- Queue depth, wait time and failed job count (Horizon).
- Reverb connections and broadcast latency.
- PostgreSQL connections, slow queries and lock waits.
- Redis memory, evictions and hit rate.
- Search indexing lag, when search moves beyond PostgreSQL.
- Container health and resource saturation.

Alerts: payment failures, auction-close failures, high bid rejection rates, queue backlog, database saturation, failed webhook processing and security events.

Logs are structured JSON from Laravel (Monolog JSON formatter), with `correlation_id` on every line. Logs never contain passwords, tokens, card data or unnecessary personal data.

## 10. Backups and recovery

Define with the business owner:
- Recovery Point Objective (RPO) and Recovery Time Objective (RTO).
- Backup frequency and retention.
- Encrypted, cross-account or isolated backup copies.
- Restore-test frequency (at least quarterly, documented).
- Disaster-recovery owner and runbook.

A backup is not considered reliable until a restore has been tested.

## 11. Release and rollback

- Feature flags (Laravel Pennant or config flags) gate high-risk capabilities such as live bidding and payments.
- Deploy progressively where possible.
- Keep schema changes backward compatible across two releases.
- Monitor for 30 minutes after each production release, with a named on-call owner.
- Roll back application images when safe. Prefer forward fixes for data migrations.
- Keep an incident communication process (see document 10 and the incident response runbook).
