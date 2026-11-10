.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).

BNPP - Test Plan for HTTP/ TLS metrics (Gateway API) And log export for IBM ROK Platform
========================================================================================

Test cases for exporting HTTP and TLS profile metrics to Prometheus and Fluentd logs to Kafka over mTLS, for BIG-IP Next for Kubernetes (BNK) 2.4.0 on Red Hat OpenShift on IBM Cloud (ROKS).

Layout
------

Each test case has its own ``TC-NN_<title>/`` folder containing:

- ``README.rst``: priority, pre-conditions, steps and verification.
- One YAML file per Kubernetes resource the test case declares, numbered in the order the test plan declares them, for example ``02-certificate-kafka-server-cert.yaml``.

Test cases that declare no resources have only a ``README.rst``.

Applying a test case's resources
--------------------------------

.. code-block:: bash

   kubectl apply -f TC-06_validate-that-http-profile-metrics-are-collected-from-big/

``kubectl`` applies a folder's files in name order. That is not always a working order: in TC-01, ``02-certificate-kafka-server-cert.yaml`` targets the ``kafka`` namespace that ``03-namespace-kafka.yaml`` creates, so apply ``03-namespace-kafka.yaml`` first.

The custom resources need their CRDs installed first: cert-manager (``Certificate``, ``ClusterIssuer``), Strimzi (``KafkaTopic``), the Logging operator (``ClusterFlow``, ``ClusterOutput``) and BNK observability (``ScrapeClass``, ``ScrapeTemplate``, ``Scrape``).

Differences from the test plan
------------------------------

The YAML in these test cases does not match the source test plan; each one's ``README.rst`` explains why and what still needs confirming:

- `TC-07_validate-that-server-ssl-and-client-ssl-profile-metrics-are <TC-07_validate-that-server-ssl-and-client-ssl-profile-metrics-are/README.rst>`__

Plan details
------------

.. list-table::
   :widths: 25 75

   * - EPIC
     - BIPNXTSPPM-1123 - HTTP/ TLS metrics (Gateway API) And log export
   * - PDR Lite
     - (blank)
   * - Initiative
     - F5-242BNK for BNPP IBM Cloud Deployment
   * - QA Status
     - IN PROGRESS
   * - Team
     - Charminar
   * - Testrail Link
     - TBD
   * - Review Date
     - (blank)
   * - Priority
     - P1
   * - Dependencies
     - Need to get/Installed Evertrust PKI, Kafka licensed version if required,  Need to get integrated to IBM ROK cluster in AWS as where Infoblox is installed. Note: if EverTrust PKI is not available we will go with Vault ACME CA
   * - Assumptions
     - Stage4 Team has been covered all the metrics validation in Datkube environment
   * - Target Release
     - BNK 2.4.0
   * - Applicable Platform
     - BNK for BNPP - Red Hat OpenShift on IBM Cloud
   * - Objective
     - Enable BNPP's Requirement to export logs from fluentd to kafka with mTLS and Metrics Export to Prometheus

Test cases
----------

.. list-table::
   :header-rows: 1
   :widths: 8 62 10 20

   * - #
     - Test case
     - Priority
     - CR files
   * - 1
     - `Verify FLO-managed Fluentd can forward logs to Kafka using custom ConfigMap and Secret on IBM ROK Platform <TC-01_verify-flo-managed-fluentd-can-forward-logs-to-kafka-using/README.rst>`__
     - P0
     - 15
   * - 2
     - `Verify logs exported to Kafka in correct JSON format <TC-02_verify-logs-exported-to-kafka-in-correct-json-format/README.rst>`__
     - P1
     - 0
   * - 3
     - `Verify Observer scrapes IPAM metrics directly from IPAM pod via gRPC sidecar <TC-03_verify-observer-scrapes-ipam-metrics-directly-from-ipam-pod/README.rst>`__
     - Not set
     - 0
   * - 4
     - `Negative: Kafka mTLS Failure with Invalid/Expired Evertrust Certificate <TC-04_negative-kafka-mtls-failure-with-invalid-expired-evertrust/README.rst>`__
     - P1
     - 0
   * - 5
     - `Negative Scenarios <TC-05_negative-scenarios/README.rst>`__
     - Not set
     - 0
   * - 6
     - `Validate that HTTP profile metrics are collected from BIG-IP TMM instances and exported to Prometheus via OTEL Collector (number_reqs, get_reqs, post_reqs, resp_2xx_cnt, resp_5xx_cnt, number_reqs) <TC-06_validate-that-http-profile-metrics-are-collected-from-big/README.rst>`__
     - P1
     - 3
   * - 7
     - `Validate that server SSL and client SSL profile metrics are collected from BIG-IP TMM instances and exported to Prometheus via OTEL Collector (tmm_profile_clientssl_stat, tls_cipher, tls_version) <TC-07_validate-that-server-ssl-and-client-ssl-profile-metrics-are/README.rst>`__
     - P1
     - 2
   * - 8
     - `Verify Fluentd Pod Restart During Active Log Export <TC-08_verify-fluentd-pod-restart-during-active-log-export/README.rst>`__
     - Not set
     - 0
   * - 9
     - `Verify OTEL Collector Pod Restart During Active Metric Export <TC-09_verify-otel-collector-pod-restart-during-active-metric/README.rst>`__
     - Not set
     - 0
