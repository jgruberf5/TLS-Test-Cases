.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-03: Verify Observer scrapes IPAM metrics directly from IPAM pod via gRPC sidecar
===================================================================================

:Test case: 3
:Priority: Not set
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

None stated in the test plan.

Steps
-----

1. Verify IPAM pod is running with gRPC metrics server
2. Apply Observer descriptor ConfigMap with IPAM metrics definition
3. Apply ObserverConfig with ipam-metrics collection job (interval: 30s)
4. Create BNK Gateway resources to trigger IPAM allocations
5. Wait 60 seconds (2 collection intervals)
6. Check Observer logs for successful IPAM metric scraping
7. Query Prometheus for IPAM metrics

Verification
------------

- Verify Observer successfully connects to IPAM gRPC server (port 50051)
- Verify IPAM metrics appear in Prometheus within 90 seconds
- Verify ``ipam_prefix`` labels show correct CIDR notation for all managed prefixes
- Verify ``ipam_prefix_scope_capacity`` shows correct IPv4 address counts
- Verify ``bnk_infoblox_connection`` gauge shows 1 (connected) when Infoblox is reachable
- Verify Metrics accurately reflect IPAM allocations (compare with ``kubectl get ipaddresses``)
- Verify No gRPC connection failures in Observer logs
- Verify Metrics labeled correctly with prefix, address_family, and gateway information

Custom resources
----------------

The test plan declares no custom resources for this test case.
