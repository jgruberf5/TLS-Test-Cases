.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-02: Verify logs exported to Kafka in correct JSON format
===========================================================

:Test case: 2
:Priority: P1
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

- Observer collecting metrics
- Fluentd forwarding to Kafka
- Sample messages available in Kafka topics

Steps
-----

1. Generate HTTP traffic (20 requests)
2. Wait for logs to appear in Kafka topic
3. Consume sample messages from Kafka
4. Validate JSON schema

Verification
------------

- Verify fields @timestamp, log or message (string with metric data) will be available JSON Format
- Verify kubernetes metadata (pod_name, namespace_name) field is available in kafka messages
- Verify stream (stdout/stderr) attribute will be available in Kafka messags

Custom resources
----------------

The test plan declares no custom resources for this test case.
