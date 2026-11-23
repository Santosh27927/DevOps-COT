# Development infrastructure and deployment architecture

Status: planning baseline  
Updated: 2026-11-23

## Confirmed decisions

- Start with local development and physical Android-device testing.
- Keep initial infrastructure small and inexpensive.
- Cloud provider, region, and budget remain undecided.
- Preserve a path to larger deployments.
- Parameterize infrastructure sizing through modules.
- Use separate frontend, backend, and DevOps repositories.
- Add iOS build and testing support later.

## Initial deployment model

Run locally using Docker Compose:

- Game service.
- Trading service.
- Market-data service when required.
- PostgreSQL.

Recorded scenarios allow testing without a live market connection.

Use separate processes or containers for runtime responsibilities.
Apply resource limits appropriate to the development computer.

This provides process separation while sharing one host.
It does not provide host-level high availability.

## First optional hosted environment

After local validation, evaluate a small single-host deployment.

A small virtual machine may host the same containers if its measured
CPU, memory, and storage requirements permit.

Provider and instance size are selected after measurement.
No particular instance family or monthly cost is committed.

Managed database and container services remain future deployment options.

## Structure

```text
devops/
├── local/
│   ├── compose/
│   └── environment-examples/
├── infra/
│   ├── modules/
│   │   ├── networking/
│   │   ├── compute-host/
│   │   ├── storage/
│   │   └── service-runtime/
│   └── environments/
│       ├── local/
│       ├── development/
│       ├── staging/
│       └── production/
├── deployments/
│   ├── game-service/
│   ├── trading-service/
│   └── market-data-service/
├── pipelines/
├── ops/
│   ├── dashboards/
│   ├── alerts/
│   └── runbooks/
└── docs/
```

Environment directories are planned boundaries.
They do not require provisioning all environments immediately.

## Parameterized infrastructure

Use typed Terraform module inputs for applicable infrastructure choices:

Host:

- Instance type.
- CPU architecture.
- Disk size.
- Image/operating-system reference.

Service:

- Image digest.
- CPU and memory limits.
- Ports and health-check settings.
- Replica count when the deployment supports replication.

Storage and database:

- Volume sizing.
- Backup destination and retention.
- Connection limits.
- Managed database sizing when introduced.

Network:

- Allowed development addresses or CIDRs.
- Public exposure and TLS settings.
- Environment-specific endpoints.

Keep provider-specific details inside provider-specific modules.
A common variable name does not make different cloud platforms
interchangeable.

## Infrastructure versus runtime configuration

Terraform variables configure infrastructure through plan/apply.

They are separate from application runtime values such as:

- Starting practice balance.
- Lesson duration.
- Opponent policy.
- Trading rules.
- Feature availability.

Application settings use the backend's validated configuration workflow.
Secrets use the selected environment's secret mechanism.

Instance resizing is an infrastructure change.
It may restart or replace resources depending on the provider and resource.

Automatic scaling requires a separate policy and compatible application
design. A replica-count variable alone does not implement autoscaling.

## Local mobile networking

Phones connect to the reachable address of the development backend.

Document:

- Backend bind address and ports.
- Development computer's network address.
- Firewall restrictions.
- Device connectivity checks.
- Development transport and certificate setup.

Avoid exposing local databases or development services publicly.

## Repository and artifact responsibilities

Frontend repository:

- Builds and validates Android artifacts.
- Later adds the iOS build/signing workflow.

Backend repository:

- Builds runtime images.
- Validates application behavior.
- Publishes versioned contracts.

DevOps repository:

- Deploys identified artifacts.
- Supplies environment configuration and secret references.
- Manages infrastructure and recovery procedures.

Record application, contract, and configuration versions per deployment.

## Phoenix integration operations

Configure Phoenix API, stream, and RPC endpoints explicitly where needed.
Keep endpoint purposes and execution permissions distinct.

Initial testing uses paper execution.
Public market-data access may still use live Phoenix endpoints.

Real transaction signing and submission remain outside the initial scope.
Avoid introducing production trading credentials into local paper tests.

Pin the SDK through the backend's dependency management.
Observe request failures, rate limiting, freshness, and reconnect behavior.

LiteSVM integration tests, if adopted, require a separate artifact/fixture
setup and may have different resource requirements.

## Observability and recovery

Start with structured logs and basic health checks.

Add dashboards and alerts as hosted testing begins.

Track:

- Service availability and errors.
- Host memory, CPU, and disk.
- Database health.
- Market-data freshness.
- Event-processing lag.
- Settlement and reconciliation failures.
- Configuration and release versions.

Preserve scenario identifiers and diagnostic references.

Before relying on hosted data:

- Configure backups.
- Test restoration.
- Document service restart and deployment recovery.
- Confirm replay and duplicate-event handling.

## Scaling progression

1. Local computer and test phones.
2. Optional small hosted development instance.
3. Separate selected workloads when measured resource pressure requires it.
4. Introduce managed storage/database or multiple hosts when justified.
5. Add scaling and stronger availability guarantees after defining targets.

Keep runtime interfaces stable while changing deployment topology.

## Limitations

- Single-host deployments have a shared host failure boundary.
- Container limits do not create independent hardware availability.
- Small instances may be unsuitable for heavy replay or protocol tests.
- Local network tests do not validate public internet behavior.
- Provider-specific modules will be required after cloud selection.
- Resizing may involve downtime or replacement.
- Stateful migration requires backup, transfer, and verification.
- Replicated services require coordinated state ownership and event delivery.
- Logging, storage, bandwidth, backups, and build services can create costs
  beyond the instance itself.
- No cloud cost, capacity, or availability guarantee is established.
- No infrastructure or pipelines have been provisioned by this document.

## Remaining decisions

- Cloud provider and region.
- Monthly budget.
- Initial concurrent-device/session target.
- First hosted deployment date.
- Backup and recovery objectives.
- Public-access and authentication requirements.
- Availability requirements beyond local testing.

## References

- [Docker Compose](https://docs.docker.com/compose/)
- [Terraform module variables](https://developer.hashicorp.com/terraform/language/values/variables)
- [Phoenix streaming practices](https://docs.phoenix.trade/sdk/ws-api-best-practices)
- [iOS export requirements](https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_ios.html)
