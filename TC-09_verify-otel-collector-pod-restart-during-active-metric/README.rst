.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-09: Verify OTEL Collector Pod Restart During Active Metric Export
====================================================================

:Test case: 9
:Priority: Not set
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

None stated in the test plan.

Steps
-----

1. Verify Observer is actively collecting metrics and exporting to OTEL Collector
2. Generate continuous HTTP traffic (20 requests over 2 minutes)
3. Query Prometheus baseline metric value: ``bnpp_http_number_reqs``
4. Restart OTEL Collector pod: ``kubectl rollout restart deployment/otel-collector -n f5-common``
5. Continue generating traffic during restart
6. Observe metric buffering behavior in Observer
7. Wait for new OTEL Collector pod to reach Running state
8. Verify OTEL Collector reconnects to Prometheus and resumes export
9. Query Prometheus final metric value
10. Calculate expected vs actual metric delta (should match traffic count ±5%)

Verification
------------

- Graceful pod termination and startup
- Metric buffering in Observer during restart
- Reconnection to Prometheus exporter
- Metric continuity (delta matches traffic ±5%)
- No time series gaps in Prometheus

Custom resources
----------------

The test plan declares no custom resources for this test case.
