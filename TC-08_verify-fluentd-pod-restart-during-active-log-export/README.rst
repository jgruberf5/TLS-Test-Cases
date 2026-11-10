.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-08: Verify Fluentd Pod Restart During Active Log Export
==========================================================

:Test case: 8
:Priority: Not set
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

None stated in the test plan.

Steps
-----

1. Verify Observer is actively collecting metrics and generating logs
2. Generate continuous HTTP traffic (100 requests over 2 minutes)
3. Monitor Kafka topic message count (baseline)
4. Restart Fluentd pod: ``kubectl rollout restart deployment/fluentd -n logging``
5. Wait for pod to terminate and new pod to start
6. Observe log buffering during restart window (30-60 seconds)
7. Wait for new Fluentd pod to reach Running state
8. Verify Fluentd reconnects to Kafka and drains buffer
9. Compare final Kafka message count with expected count
10. Check for any lost log messages during restart

Verification
------------

- Graceful pod termination and startup
- Log buffering in Observer during restart window
- Reconnection to Kafka with mTLS
- Buffer drainage after recovery
- Zero message loss during 30-60 second restart downtime

Custom resources
----------------

The test plan declares no custom resources for this test case.
