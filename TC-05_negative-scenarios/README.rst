.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-05: Negative Scenarios
=========================

:Test case: 5
:Priority: Not set
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

- Kafka cluster with mTLS configured (port 9094)
- Fluentd deployed with ClusterOutput to Kafka
- Observer deployed with metric collection jobs
- Valid mTLS certificates in toda-kafka-client-tls secret
- Backup of working configurations

Steps
-----

**Scenario 4: Missing Certificate Secret**

1. Delete f5-toda-fluentd-custom-secret secret (``kubectl delete secret toda-kafka-client-tls -n logging``)
2. Trigger Fluentd pod restart (``kubectl rollout restart deployment/fluentd -n logging``)
3. Check pod status and events
4. Verify pod reports volume mount error (ContainerCreating state)
5. Inspect pod describe output for mount failure
6. Recreate secret with valid certificates
7. Verify pod enters Running state
8. Confirm log forwarding resumes

**Scenario 5: Invalid Observer Table Name**

1. Edit Observer descriptor ConfigMap
2. Add non-existent table name (e.g., ``profile_invalid_stat``)
3. Apply updated ConfigMap and restart Observer pod
4. Check Observer logs for table query errors
5. Verify Observer logs error: "Table profile_invalid_stat not found"
6. Confirm Observer continues processing valid tables (HTTP, TCP, VS, Pool)
7. Verify metrics from valid tables still exported
8. Remove invalid table from descriptor and restart

Verification
------------

**Scenario 4 - Missing Secret**:

- Verify Pod status: ``ContainerCreating`` (not Running)
- Verify Pod events: ``Warning FailedMount: Unable to mount volumes... secret "toda-kafka-client-tls" not found``
- Verify Pod describe shows volume mount error
- Verify After secret recreation: Pod transitions to Running
- Verify Fluentd starts and connects to Kafka successfully

**Scenario 5 - Invalid Table Name**:

- Verify Observer logs: ``ERROR: relation "profile_invalid_stat" does not exist`` or ``Table not found: profile_invalid_stat``
- Verify Observer continues execution (does not crash)
- Verify Valid tables (profile_http_stat, profile_tcp_stat, virtual_server_stat, pool_stat) still collected
- Verify Kafka topics (bnpp-http-logs, bnpp-tcp-logs) still receive metrics from valid tables
- Verify Prometheus still shows metrics from valid tables
- ❌ NO crash or pod restart

Custom resources
----------------

The test plan declares no custom resources for this test case.
