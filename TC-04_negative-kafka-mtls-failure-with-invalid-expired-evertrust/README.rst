.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-04: Negative: Kafka mTLS Failure with Invalid/Expired Evertrust Certificate
==============================================================================

:Test case: 4
:Priority: P1
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

**Scenario 1: Expired Certificate Handling**

1. Generate expired mTLS certificate (valid until yesterday)
2. Update f5-toda-fluentd-custom-secret secret with expired cert
3. Restart Fluentd pod
4. Observe Fluentd logs for TLS handshake errors
5. Generate 50 test log messages
6. Verify buffering and retry behavior
7. Restore valid certificate
8. Confirm message delivery after recovery

**Scenario 2: Invalid CA Certificate**

1. Generate new self-signed CA certificate (not trusted by Kafka)
2. Update ca.crt in f5-toda-fluentd-custom-secret secret
3. Restart Fluentd pod
4. Attempt to send logs to Kafka
5. Observe connection rejection and error logging
6. Verify Fluentd does not crash or enter CrashLoopBackOff
7. Restore correct CA certificate
8. Verify reconnection and message delivery

**Scenario 3: Unreachable Kafka Broker**

1. Scale Kafka StatefulSet to 0 replicas (``kubectl scale sts kafka-cluster-kafka -n kafka --replicas=0``)
2. Generate 100 test log messages
3. Monitor Fluentd buffer directory size (``/var/log/fluentd-buffers/``)
4. Observe retry attempts in Fluentd logs (exponential backoff)
5. Scale Kafka back to 1 replica
6. Wait for broker readiness
7. Verify Fluentd drains buffered messages
8. Confirm no message loss

Verification
------------

**Scenario 1 - Expired Certificate**:

- Verify Fluentd logs: ``SSL_connect: certificate verify failed (certificate has expired)``
- Verify Messages buffered to ``/var/log/fluentd-buffers/`` (file buffer)
- Verify Retry attempts logged with exponential backoff
- Verify After cert restoration: All buffered messages delivered to Kafka
- ❌ NO message loss

**Scenario 2 - Invalid CA**:

- Verify Fluentd logs: ``SSL certificate problem: self signed certificate in certificate chain``
- Verify Connection rejected by Kafka broker
- Verify  Fluentd pod remains in Running state (not CrashLoopBackOff)
- Verify Error logged but plugin continues retry loop
- Verify After CA restoration: Connection established, messages delivered

**Scenario 3 - Unreachable Broker**:

- Verify Fluentd logs: ``Connection refused - connect(2) for "kafka-cluster-kafka-bootstrap.kafka.svc.cluster.local" port 9094``
- Verify Buffer directory size increases (messages accumulating)
- Verify Retry attempts with backoff: 1s, 2s, 4s, 8s, 16s, 32s...
- Verify After broker recovery: Buffer drains (messages sent to Kafka)
- Verify Kafka topic message count matches sent count (100 messages)
- ❌ NO message loss

Custom resources
----------------

The test plan declares no custom resources for this test case.
