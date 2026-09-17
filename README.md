# cloud data security: the controls that matter, the mistakes that actually leak data, and how to pick infrastructure that helps

Most people who search for cloud data security aren't looking for a philosophy lecture. They want to know three things: what actually causes cloud data leaks, what they're responsible for versus what their provider handles, and what to look for when they're picking (or reconsidering) a cloud platform.

This guide covers all three. It walks through the failure points that show up again and again in real incidents, gives you a practical control checklist you can audit your own setup against, and then looks at how an infrastructure provider like Sharktech — an OpenStack-based cloud host with data centers in Los Angeles, Las Vegas, Denver, Chicago, and Amsterdam — handles the provider side of the equation, including current plan pricing pulled from their live order system.

## What "cloud data security" actually means

Strip away the marketing and cloud data security comes down to protecting three things about your data:

- **Confidentiality** — nobody who shouldn't see your data can see it
- **Integrity** — your data doesn't get altered or corrupted without you knowing
- **Availability** — your data is there when you (or your customers) need it

That third one gets skipped surprisingly often in security discussions, which is odd considering a DDoS attack that takes your database offline for two days is very much a data security problem. We'll come back to that.

The other thing worth understanding up front: cloud data security is not one product you buy. It's a stack of controls — encryption, access management, network isolation, backups, monitoring — layered on top of infrastructure whose security is split between you and your provider.

## The shared responsibility model: the #1 thing to get right

Every major cloud provider — AWS, Azure, Google Cloud, and smaller hosts alike — operates on some version of the shared responsibility model. The split works roughly like this:

**The provider secures the cloud itself**: physical data centers, hardware, hypervisors, the network backbone, and the underlying infrastructure availability.

**You secure what you put in the cloud**: your data, your user accounts and permissions, your applications, your firewall rules, your storage bucket configurations.

This is where a huge share of cloud incidents originate. Industry roundups of breach statistics (SentinelOne and Exabeam both publish running tallies) consistently find that a large fraction of cloud breaches trace back to customer-side misconfiguration — open storage, excessive permissions, forgotten public endpoints — rather than the provider's infrastructure being cracked. Gartner survey data, widely cited across security publications, has attributed the overwhelming majority of cloud security failures to misconfiguration rather than provider compromise.

The practical takeaway: moving to the cloud does not outsource your data security. It changes which parts you're responsible for. If you take one thing from this article, take that.

## Where cloud data actually leaks: the usual suspects

### Misconfigured storage and exposure

The classic. A storage bucket or database gets set to public "just for testing," the project ships, and the data sits exposed for months. Regular scanning for misconfigured resources — public access controls, weak encryption settings, exposed services — is standard advice from posture-management vendors like Wiz for exactly this reason. If you're on a platform where you manage the firewall and access rules yourself (which is most IaaS clouds, including OpenStack-based ones), you own this layer completely.

### Weak identity and access management

Shared admin logins, no multi-factor authentication, permissions granted broadly "to keep things moving" and never revoked. Identity hardening — least-privilege access, MFA, short-lived credentials where possible — appears at or near the top of virtually every cloud security best-practices list from Sysdig, Fidelis, and AWS's own SMB guidance.

### Data crossing boundaries unencrypted

Data at rest (sitting in storage, databases, backups) and data in transit (moving between services, users, and regions) need different protection, and both need to be encrypted. Google's cloud documentation treats encryption in transit and at rest as foundational, separate controls — because they are. A workload with encrypted disks but unencrypted inter-service traffic is only half protected.

### Availability attacks

DDoS remains one of the cheapest ways to attack a business, and the legal stakes are real — Sharktech's own DDoS resource pages note that under the US Computer Fraud and Abuse Act, attack perpetrators can face up to 10 years and $500,000 in fines. Your legal recourse doesn't restore your revenue while your service is down, though. Availability protection belongs in your data security plan, not in a separate "network stuff" folder.

### Lock-in and data hostage situations

An underrated one. If your provider makes it expensive or impossible to export your data and VM images, you've got a data ownership problem — which becomes a business continuity problem the day pricing changes or the provider pivots. High egress fees are the most common version of this trap; Sharktech explicitly calls out egress costs on their cloud FAQ as a lock-in mechanism used by large providers.

## A practical cloud data security checklist

Based on the consistent recommendations across current best-practice guides (Sysdig's 2026 list, Fidelis's top-10, Wiz's data security guidance), here's the condensed version you can actually audit against:

1. **Map your responsibility boundary.** Write down what your provider covers and what you cover. Revisit it when you add services.
2. **Encrypt at rest and in transit.** Verify both, separately. Don't assume.
3. **Harden IAM.** Least privilege, MFA everywhere, no shared credentials, regular access reviews.
4. **Scan for misconfiguration continuously.** Public buckets, open ports, permissive security groups — automate the checks rather than doing annual manual audits.
5. **Isolate network traffic.** Put backend services on private networks so they're not exposed to the public internet at all.
6. **Back up, and test restores.** A backup you've never restored is a hope, not a control.
7. **Monitor everything.** Traffic analytics, usage records, anomaly alerts.
8. **Protect availability.** DDoS mitigation and redundancy are part of data security.
9. **Keep an exit path.** Make sure you can export your data and images when you want to.

Items 5 through 9 are where your choice of infrastructure provider does the heavy lifting. Which brings us to the provider side.

## How Sharktech's cloud platform handles the provider half

Sharktech has been around since 2003 and runs its own autonomous system (AS46844) with peering at major internet exchange points — they describe themselves as an ISP that also sells hosting, which shapes how their security is built. Here's what's relevant to cloud data security specifically, based on their current official pages:

**Network isolation is built into the platform.** Their OpenStack-based cloud includes private networking (backend traffic between VMs stays off the public internet), security groups with granular firewall rules, virtual routers with NAT, and load balancers. If you're running the checklist above, items 5 and 6 map directly onto features in the management panel.

**DDoS protection is included, not an add-on.** Every hosted service — cloud, VPS, dedicated — ships with always-on DDoS mitigation as standard, upgradeable in 100Gbps increments. Third-party reviews put the standard protection at 60Gbps per IP, with enterprise deployments scaling to 1Tbps. For context, most volumetric attacks that actually knock services offline fall in the 5–20Gbps range, so the baseline protection covers the common-case attack outright. Their mitigation stack filters the usual families: UDP floods, HTTP floods, SYN floods, DNS/NTP/SSDP amplification, Slowloris, and so on.

**No vendor lock-in — with teeth.** This is the part that separates marketing claims from policy. On Sharktech's cloud you can download your server disk images whenever you want, via the portal or API, for offsite backup or migration. You can also upload your own ISOs and images. Combined with free inbound traffic and outbound overage at $0.002/GB, the exit cost stays rational. For data ownership purposes, that's the difference between "your data is always yours" as a slogan and as an operational fact.

**Patching cadence.** They use official Linux cloud images from the major distributions, refreshed weekly, so instances deploy on current patches — and you can layer cloud-init scripts on top for your own hardening automation.

**Billing is separated from infrastructure management.** Their cloud portal isolates financial operations from the infrastructure control plane, which reduces cross-system attack surface and lets you split billing and technical access between different team members.

**Availability guarantees.** 99.999% uptime guarantee, fully redundant hyper-converged infrastructure with automatic failover, and five data center locations (Los Angeles, Las Vegas, Denver, Chicago, Amsterdam) so you can place workloads geographically — which matters both for latency and for data residency considerations. Note that data residency and compliance certification (GDPR, HIPAA, and so on) are things you'll need to verify against your specific regulatory situation; Amsterdam gives you an EU foothold, but compliance is workload-specific and shared-responsibility applies here too.

**Independent testing.** HostAdvice's hands-on review of the public cloud measured ticket responses in under 40 minutes (their test ticket went in at 1:11 AM and got a reply at 1:50 AM), roughly 45.5 GB/sec memory throughput, ~10Gbps network performance from the test VM, and gave the service an overall 9.4/10 — while also flagging the honest downsides: no money-back guarantee (all payments non-refundable, with a 30-day window for billing disputes that resolve in account credit, not cash) and a limited number of regions compared to hyperscalers. Their benchmark data also showed the standard SSD tier at traditional-SSD speeds while the NVMe tier hit ~5,020 MB/s sequential reads — so for I/O-heavy workloads, the storage tier choice matters.

## Sharktech cloud plans and pricing (current order-page data)

The following reflects what's live on Sharktech's order system right now. Public Cloud is the pay-as-you-go line: each tier includes a committed resource pool, and consumption above the commit bills hourly (CPU at $0.0025/hr per core, RAM at $0.0035/hr per GB, NVMe at $0.00009/hr per GB, SSD at $0.00006/hr per GB, HDD at $0.00002/hr per GB). Tiers below Enterprise carry a maximum resource cap so a runaway workload can't produce a runaway invoice.

| Plan | vCPU | RAM | SSD Storage | Included Bandwidth | Starting Price | Billing | Order |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Small | 4–16 | 8–32 GB | 300–2,400 GB | 20 TB | $39.00/mo | Monthly + hourly overage | [ Deploy the Small tier](https://portal.sharktech.net/aff.php?aff=1611&pid=602) |
| Medium | 8–32 | 16–64 GB | 800–6,400 GB | 20 TB | $79.00/mo | Monthly + hourly overage | [ Deploy the Medium tier](https://portal.sharktech.net/aff.php?aff=1611&pid=602) |
| Large | 32–128 | 64–256 GB | 1,500–12,000 GB | 20 TB | $249.00/mo | Monthly + hourly overage | [ Deploy the Large tier](https://portal.sharktech.net/aff.php?aff=1611&pid=602) |
| Enterprise | 64+ (uncapped) | 128+ GB (uncapped) | 5,000+ GB (uncapped) | 20 TB | $499.00/mo | Monthly + hourly overage | [ Deploy the Enterprise tier](https://portal.sharktech.net/aff.php?aff=1611&pid=602) |
| Custom | Custom | Custom | Custom | Custom | Quote | Custom | [ Get a custom cloud quote](https://bit.ly/SharKTech) |

A few details worth knowing before you check out:

- The order page defaults to Los Angeles; the same tiers are orderable in Las Vegas, Denver, Chicago, and Amsterdam.
- Inbound traffic is unlimited and free. Outbound beyond the included allowance bills at $0.002/GB (the plan listings show 20 TB included; Sharktech's cloud service notes also cite a 5,000 GB baseline for cloud services with the same overage rate — confirm the exact allowance for your configuration at checkout).
- The first public IPv4 address is free; additional ones are $1.50/month each.
- Payments: credit card, PayPal, wire transfer, Western Union, and Alipay.
- Everything is non-refundable, so size your first deployment conservatively.

Sharktech also runs two adjacent product lines worth knowing about if the Public Cloud tiers are more than you need. **Smart VPS** (Proxmox-based, NVMe-backed, 60Gbps DDoS protection, from $7.95/month with 50% off on annual billing — the entry tier works out to about $3.98/month) gives you a resource pool you can split into multiple VMs, and it's the cheapest sane way to evaluate their network before committing to a cloud plan: [👉 Try a Smart VPS from $7.95/month](https://bit.ly/SharKTech). **Dedicated Cloud** is the prepaid fixed-rate version of the same infrastructure — you get exactly the resources you order for a flat monthly fee (8–512 vCPU, 16–1024 GB RAM, from $86.23/month), which is the right shape for teams that need predictable invoices: [👉 Ask about Dedicated Cloud pricing](https://bit.ly/SharKTech).

## Matching plans to actual data security needs

A quick editorial read on how the tiers map to different situations, based on the configurations above:

- **Small ($39/mo)** fits staging environments, small production apps, and getting familiar with the OpenStack panel. Four cores and 8 GB covers a lot more than people expect when it's not shared with neighbors.
- **Medium ($79/mo)** is the sweet spot for a small team's production workload — enough headroom to separate a database tier from app servers on a private network without thinking twice.
- **Large ($249/mo)** is for traffic-heavy or multi-service deployments where you want meaningful NVMe capacity and multiple VMs with isolated network segments.
- **Enterprise ($499/mo, uncapped)** is the tier where the resource cap disappears, so it's the one that demands actual monitoring discipline — pair it with the usage records in the panel.
- **Custom** for anything with unusual compliance, storage, or throughput requirements; sales will scope it.

Whichever tier you land on, the customer-side controls from the checklist earlier still apply. The platform gives you private networks, security groups, weekly-patched images, and included DDoS mitigation; using them well is on you.

## Cloud data security FAQ

**Is my data safe just because the provider has good security?**
No. The shared responsibility model means the provider's DDoS protection, redundant infrastructure, and patching cover the platform — while your data exposure depends on your firewall rules, access management, and storage configuration. Most cloud breaches trace back to customer-side misconfiguration, not cracked infrastructure.

**Does encryption matter if my provider "handles security"?**
Yes, doubly. Verify encryption at rest for your volumes and databases, and encryption in transit for traffic between your services — especially anything crossing between private and public network segments. These are separate settings and both need to be on.

**How does DDoS protection relate to data security?**
Availability is one of the three pillars of data security. An attack that takes your service offline makes your data unavailable, which is a security failure even if nothing is exfiltrated. Sharktech includes always-on mitigation on every plan, which removes the most common excuse for skipping this.

**What's the deal with egress fees and data security?**
Egress pricing is the quiet lock-in mechanism in cloud hosting — if moving your data out costs a fortune, you've effectively lost ownership flexibility. Look for providers with free inbound traffic, low published overage rates, and the ability to export full disk images on demand. Sharktech checks all three boxes.

**Can I test Sharktech before committing serious workloads?**
There's no free trial and no refunds, but hourly overage billing means a small deployment costs very little to run for a week, and Smart VPS at $7.95/month (about $3.98/month on annual billing) is an even cheaper on-ramp to evaluate the network and panel first.

**Which locations can I deploy in?**
Los Angeles, Las Vegas, Denver, Chicago, and Amsterdam — selectable during deployment, which lets you keep data geographically close to its users or within an EU jurisdiction where that matters.

## The bottom line

Cloud data security is two jobs, not one. The provider's job is a hardened, redundant, DDoS-protected platform with sane isolation tools and no exit traps — and on the evidence of their current setup, Sharktech does that job well, with included DDoS mitigation, private networking, weekly-patched images, downloadable disk images, and honest pricing from $39/month for the OpenStack public cloud. Your job is the configuration layer: encryption, least-privilege access, misconfiguration scanning, tested backups.

The breaches that make headlines are almost never the provider's infrastructure failing. They're the open bucket, the shared password, and the forgotten public endpoint. Fix your half of the responsibility split, and pick a provider whose half doesn't fight you — [👉 see Sharktech's current cloud plans and pricing](https://portal.sharktech.net/aff.php?aff=1611&pid=602) if the infrastructure side of that equation sounds like what you're after.
