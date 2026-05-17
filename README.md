# Wazuh-Deployment-and-Service-Recovery-LA


## Troubleshooting Wazuh Dashboard and Indexer Startup Problems in a Virtual Environment

---

# 📌 Project Overview

This repository documents a practical lab setup involving the deployment of the Wazuh SIEM platform inside a virtual machine environment.

During deployment, several backend services failed to initialize correctly, causing the dashboard to become inaccessible and the indexer service to remain unstable.

The purpose of this project is to document:

- Common deployment issues in Wazuh
- Service dependency problems
- OpenSearch indexer startup failures
- Resource optimization techniques
- Recovery and troubleshooting procedures

---

# 🖥️ Lab Environment

| Component | Information |
|---|---|
| Platform | Virtual Machine |
| Virtualization Software | VMware Workstation |
| Guest OS | Ubuntu Server |
| Wazuh Version | 4.x |
| Initial RAM Allocation | 2 GB |
| Updated RAM Allocation | 6 GB |
| Dashboard Access Port | 443 |
| Wazuh API Port | 55000 |
| Indexer Port | 9200 |

---

# ⚠️ Issues Encountered

After completing the Wazuh installation process, the following issues appeared:

- Dashboard loading indefinitely
- Login page inaccessible
- Browser connection failures
- Delayed API responses
- Wazuh indexer service startup timeout
- OpenSearch initialization failure
- Dashboard showing empty monitoring data

Example errors observed:


🔍 Technical Analysis

The deployment issue was mainly related to insufficient virtual machine resources and incorrect service-level configurations.

Several components of Wazuh depend heavily on OpenSearch, which requires proper memory allocation and kernel tuning.

The investigation identified multiple contributing factors.


🧠 Root Causes
1. Low Memory Allocation

The virtual machine was initially configured with limited RAM, causing OpenSearch to fail during bootstrap.

2. Improper Java Heap Values

The default JVM heap allocation exceeded available system memory.

Conflicting JVM entries inside the configuration file also caused instability.

3. No Swap Space Configured

Without swap memory, the system struggled during heavy indexer initialization.

4. Kernel Limits Not Configured

The required Linux kernel parameter for OpenSearch was below the recommended threshold.

5. Service Startup Timeout

Indexer startup time exceeded the default systemd timeout limit.


🛠️ Troubleshooting and Fixes
✅ Increased VM Memory

RAM allocation was increased to improve OpenSearch performance and reduce startup delays.

✅ Optimized JVM Heap Configuration
File Modified

/etc/wazuh-indexer/jvm.options

Updated Values

-Xms1g
-Xmx1g

Unused and duplicate heap definitions were removed.

✅ Added Swap Memory

fallocate -l 2G /swapfile

chmod 600 /swapfile

mkswap /swapfile

swapon /swapfile

Swap was configured permanently through /etc/fstab.

✅ Tuned Linux Kernel Parameter
sysctl -w vm.max_map_count=262144

Persistent configuration:

echo "vm.max_map_count=262144" >> /etc/sysctl.conf

sysctl -p

✅ Increased Indexer Startup Timeout
Service Override

systemctl edit wazuh-indexer

Added Configuration


TimeoutStartSec=300

Reloaded systemd configuration:

systemctl daemon-reexec
systemctl daemon-reload

✅ Verified OpenSearch Connectivity

curl -k -u admin:admin https://localhost:9200

Successful response confirmed that the indexer service was functioning properly.

✅ Fixed Dashboard API Delay
Configuration File

/etc/wazuh-dashboard/opensearch_dashboards.yml

Updated Timeout Configuration
wazuh:
  api:
    host: "localhost"
    port: 55000
    protocol: "https"
    timeout: 60000

Dashboard service restarted afterward.


✅ Final Outcome

After applying all fixes:

Wazuh Dashboard became accessible
Indexer service stabilized
API communication restored
Login functionality working correctly
Monitoring interface loaded successfully

The dashboard initially displayed:

No results found

This behavior was expected because no monitoring agents had been enrolled yet.


📚 Key Learning Outcomes

OpenSearch requires careful memory planning
JVM tuning is critical for SIEM stability
Swap memory improves low-resource deployments
Linux kernel optimization affects indexer startup
Dashboard issues are often linked to backend service failures
Service timeout tuning can prevent false startup failures

