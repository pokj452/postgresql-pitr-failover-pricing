# Managed PostgreSQL comparison: how many days of PITR you really get, whether failover counts as HA, and where storage ceilings and per-GB pricing bite (plan table included)

Three things decide whether a managed PostgreSQL plan fits you: how many days of point-in-time recovery it keeps, whether "high availability" includes a standby node you are actually paying for, and what happens when the database outgrows the storage ceiling. Everything else, including the monthly price, is secondary to those three.

The cheapest verified entry price in this comparison is UpCloud at **$8/month** for 1 GB RAM and 10 GB storage, with a 3-day recovery window. DigitalOcean starts at **$15.15/month** with 7 days. Vultr prices by plan size rather than by tier, and the tier decides your recovery window: Hobbyist keeps none, Startup 2 days, Business 14 days, Premium 30 days.

## Quick take: which tier to start on

> If the data matters, start on Vultr's Business tier. It keeps 14 days of point-in-time recovery, supports up to three read replica nodes for failover, includes PgBouncer connection pooling, and runs as a single-tenant cluster behind a 99.99% SLA. Hobbyist has no self-service restore, so it belongs in dev and test only.
>
> 👉 [See Vultr's managed PostgreSQL plans and tier pricing](https://www.vultr.com/products/managed-databases?ref=9738262-9J)

## Vultr managed PostgreSQL plans by tier

Vultr's documentation ties recovery features to the tier you buy, which is unusual among the providers here and the single most useful thing to know before ordering. All tiers get automated backups. What changes is whether you can act on them.

| Tier | PITR window | Restore and forking | Best for | Pricing |
| --- | --- | --- | --- | --- |
| Hobbyist | None | Automated backups, no self-service restore | Disposable dev and test data | [Hobbyist tier pricing](https://www.vultr.com/products/managed-databases?ref=9738262-9J) |
| Startup | 2 days | PITR, restore, fork | Small apps with a short rollback tolerance | [Startup tier pricing](https://www.vultr.com/products/managed-databases?ref=9738262-9J) |
| Business | 14 days | PITR, restore, fork | Production apps where a bad deploy surfaces within two weeks | [Business tier pricing](https://www.vultr.com/products/managed-databases?ref=9738262-9J) |
| Premium | 30 days | PITR, restore, fork | Regulated data and long incident investigations | [Premium tier pricing](https://www.vultr.com/products/managed-databases?ref=9738262-9J) |

Vultr quotes prices per plan size, so the figure moves with the RAM and storage you select rather than with the tier name alone. The live plan page is the only place that gives you the current number for the size you want.

One inconsistency worth flagging: Vultr's product page advertises "up to 14 days of backups with 3-node clusters," while the PostgreSQL documentation lists the windows above, with Premium at 30 days. The per-tier figures come from the documentation, so confirm the window for your exact plan at checkout instead of assuming the marketing number applies.

## Point-in-time recovery: how far back can you roll?

Point-in-time recovery is what saves you when someone drops a column or runs a bad migration. A daily backup alone restores you to last night; PITR replays the write-ahead log to a chosen minute, so you can land just before the mistake.

The window varies more than the price does. DigitalOcean keeps **7 days** and lets you restore to any point inside it. UpCloud's PostgreSQL plans keep **3 days** (its MySQL plans keep 2). Vultr ranges from nothing on Hobbyist to 30 days on Premium, and its PITR is down to the minute rather than to the hour.

Consider a migration that nulls a column on Monday. Nobody notices until Thursday, when a report comes back empty. On a 3-day window the Monday state is already gone, and you are restoring from a full backup and reconciling by hand. A 14- or 30-day window covers that scenario, plus the slower version of it where the bug ships on a Friday and is found after the weekend.

Two providers in this comparison don't publish a PITR number at all: Scaleway and Fly.io both mention point-in-time restore without stating a retention period. If PITR length matters to you, that question needs an answer before you migrate, not after.

👉 [Check which Vultr tier gives you 14 or 30 days of recovery](https://www.vultr.com/products/managed-databases?ref=9738262-9J)

## Automatic failover is not the same as high availability

Almost every provider advertises "automatic failover," and it means less than it sounds. DigitalOcean states the distinction plainly: all of its clusters have automated failover, which replaces a degraded or failing node, but high availability requires redundancy on top of that, and a cluster needs at least one standby node to actually be highly available. In other words, failover is a behavior; HA is a piece of hardware you have to buy.

Heroku draws the line the same way from the other direction. HA is bundled into Premium and Shield plans, where a standby replica sits in a physically separate availability zone and takes over when the primary fails. Cheaper tiers do not get it.

Fly.io goes the opposite route and says every plan includes high availability, backups, and connection pooling. That is a cleaner promise, though it doesn't tell you how many nodes are behind it.

Vultr's documentation describes HA and failover as the same mechanism: you add up to three read replica nodes, and the platform uses health checks and leader election to promote a replica when the primary fails, then rebalances once the failed node returns. That is a sound design, but it makes the replica count the thing to check. Ask three questions before you order from anyone:

1. Is the standby or replica included in the quoted price, or billed as a separate node?
2. Does failover survive the loss of an availability zone, or only the loss of a single node?
3. How long does promotion take, and does the connection string stay stable during it?

## Storage ceilings and per-GB pricing

Storage is where the cheap-looking plan gets expensive, because most providers charge for provisioned capacity rather than for what you use.

- **Fly.io** starts at 10 GB and caps at **1 TB**, with storage replicated across every node and billed at **$0.28 per provisioned GB per month** on top of the plan fee. At that rate, 100 GB is about $28/month in storage alone, and a 1 TB database (1,024 GB) runs about $287/month before you pay for compute.
- **Scaleway** attaches Block Storage that scales to **10 TB with no downtime**, without changing node type, and bills hourly with no upfront or monthly commitment.
- **Heroku Advanced** includes 100 GB and scales to **200+ TB**, with compute and storage decoupled so you can run a huge archive on light compute or a small hot dataset on heavy compute.
- **DigitalOcean** advertises scaling to 30 TB on the product page, while the Advanced Edition FAQ on the same page says 32 TB. Pick the number that matches the edition you buy.
- **UpCloud** covers 25 to 100 GB in its published plans, with additional storage up to 200 GB.

The practical rule: below roughly 100 GB, the monthly plan price dominates and you should optimize for PITR and HA. Past a few hundred GB, the per-GB storage rate decides the bill, and a provider that looks expensive at entry can be the cheaper one at scale.

## How the main providers compare

| Provider | Entry price | PITR | Storage ceiling | HA notes |
| --- | --- | --- | --- | --- |
| UpCloud | $8/mo, $30/mo, $60/mo | 3 days (PostgreSQL) | 25–100 GB, plus up to 200 GB | Failover included; 99.999% uptime SLA claimed for HA deployments |
| DigitalOcean | $15.15/mo | 7 days | 30 TB (32 TB per Advanced FAQ) | Failover on all clusters; HA needs at least one standby node |
| Scaleway | Hourly, no commitment (rates not published) | Not stated | 10 TB block storage, resized without downtime | Stand-alone, HA, or multi-AZ read replicas |
| Fly.io | $0.28/GB/mo storage plus plan fee | Not stated | 1 TB | HA included on every plan, PgBouncer on every plan |
| Heroku | Not published on the product page | Not stated | 200+ TB (Advanced) | HA only on Premium and Shield; PCI and HIPAA require Shield |

Vultr sits outside that table because its cost depends on tier and plan size together. Its published capabilities: a 99.99% SLA, single-tenant isolation at the compute, network, and storage level, SOC 2 Type II plus HIPAA and GDPR compliance claims, private networking and firewalls, identity and access management, PostgreSQL versions 13 through the current release, PgBouncer, migration tooling, and a Terraform provider.

## What Vultr gives you, and who should look elsewhere

The compliance position is Vultr's strongest card here. Single-tenant architecture with private networking and IP whitelisting, plus SOC 2 Type II and HIPAA on the managed database page, covers workloads that would otherwise push you toward Heroku's Shield tier. Heroku publishes no price for Shield, so that comparison is worth making before you assume the compliance premium is unavoidable.

The weak spot is disclosure. Vultr doesn't publish a flat per-tier price list, and its own pages disagree on backup retention. Anyone who wants one number to compare across providers has to pull it plan by plan.

Look elsewhere if you need a single database above a few terabytes (Heroku Advanced goes to 200+ TB), if you want PITR on the cheapest possible plan (UpCloud's $8 tier keeps 3 days; Vultr's Hobbyist keeps none), or if you want HA bundled without thinking about node counts (Fly.io includes it on every plan).

👉 [Compare Vultr's managed database tiers and what each one includes](https://www.vultr.com/products/managed-databases?ref=9738262-9J)

## Which plan fits your situation

- **Side project or staging data you can rebuild:** Vultr Hobbyist, or UpCloud's $8 plan if you want 3 days of PITR for less than the price of a lunch.
- **Small production app, tight budget:** DigitalOcean at $15.15/month gives 7 days of PITR, and you can add a standby when you need real HA.
- **Production app where incidents surface slowly:** Vultr Business, 14 days of PITR plus up to three replicas.
- **Regulated data or long investigations:** Vultr Premium at 30 days, or Heroku Shield if you specifically need its compliance tier.
- **Database heading past 10 TB:** Heroku Advanced, or Scaleway if European data residency matters (Paris, Amsterdam, Warsaw).
- **Vector search or PostGIS work:** Scaleway, Heroku, Fly.io and UpCloud all list pgvector or PostGIS, and UpCloud ships 70+ pre-installed extensions including Citus and pg_partman.

Two things to confirm before you enter a card number. First, the PITR window of the exact plan you are buying, since it varies by tier and Vultr's pages don't agree. Second, whether the standby or replica is inside the quoted price or added on top, because that difference is often larger than the gap between providers.

👉 [Check current Vultr managed PostgreSQL pricing before you order](https://bit.ly/VuLtr)
