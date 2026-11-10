.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-07: Validate that server SSL and client SSL profile metrics are collected from BIG-IP TMM instances and exported to Prometheus via OTEL Collector (tmm_profile_clientssl_stat, tls_cipher, tls_version)
==========================================================================================================================================================================================================

:Test case: 7
:Priority: P1
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

- Observer deployed with profile_serverssl_stat and profile_clientssl_stat collection jobs
- OTEL Collector configured with Prometheus remote write exporter
- Prometheus server accessible
- BIG-IP TMM instances running (TMM-0, TMM-1)
- HTTPS Virtual Server with server SSL and client SSL profiles configured
- OpenSSL s_server running for TLS backend

Steps
-----

1. Start OpenSSL s_server (TLS backend) on port 8443:

   .. code-block:: bash

      openssl s_server -accept 8443 -cert server.crt -key server.key -www

2. Apply HTTPS Virtual Server CR with SSL profiles:

   - Server SSL profile: Client-facing TLS (port 443)
   - Client SSL profile: Backend TLS to OpenSSL s_server (port 8443)

3. Apply ScrapeClass, ScrapeTemplate and Scrape with table definitions:

   - profile_serverssl_stat (client-facing TLS)

4. Apply ObserverConfig with SSL collection jobs (interval: 30s)
5. Wait 60 seconds for TMM to populate SSL profile tables
6. Send HTTPS traffic from client to Gateway/Virtual Server, distributed across TMMs:

   - 10 HTTPS requests to Virtual Server (TMM-0: ~5, TMM-1: ~5)
   - Use curl with TLS 1.2 and TLS 1.3
   - Mix cipher suites (AES-GCM, ChaCha20)

7. Wait 90 seconds (3 collection intervals)
8. Query Prometheus for Client SSL metrics
9. Verify metrics aggregated across both TMM instances

Verification
------------

**Expected Results - Client SSL Metrics (Backend-Facing)**:

- Verify Prometheus shows SSL metrics with labels ``{tmm="0"}`` and ``{tmm="1"}``
- Verify Metrics present from **tmm_profile_clientssl_stat** table
- Verify ``cur_conns`` ≥ 0 (current active backend SSL connections)
- Verify ``tot_native_conns`` ≥ 10 (total backend TLS handshakes to OpenSSL s_server)
- Verify ``tot_compat_conns`` ≥ 0 (total compatibility mode backend handshakes)
- Verify ``tls_version`` metric captured in premetheus and ``tls_cipher`` (backend cipher suite - e.g., "ECDHE-RSA-AES256-GCM-SHA384")
- Verify Data aggregated: ``sum(tot_native_conns)`` = total backend handshakes

Deviation from the test plan
----------------------------

.. warning::

   The CR YAML for this test case **differs from the Confluence test plan**.

   In the test plan, TC-07's YAML is a byte-for-byte copy of TC-06's: it scrapes only
   ``profile_http_stat`` and ``virtual_server_stat``, so no SSL metric is ever collected.
   It also reused TC-06's resource names, so applying it would overwrite TC-06's
   ScrapeClass, ScrapeTemplate and Scrape.

   The replacement:

   - uses its own names (``f5-ssl-metrics-class``, ``f5-ssl-metrics-template``);
   - adds ScrapeTemplate jobs for ``profile_clientssl_stat`` and ``profile_serverssl_stat``,
     the SSL tables this test case and the BNPP design ("BNPP Metrics & Log Export to ELK")
     name;
   - drops the hand-written Scrape CR. The Observer generates the per-metric Scrape from the
     ScrapeTemplate using its descriptor, so every metric the descriptor defines for those
     tables is exported without listing metric names.

   Before relying on it, confirm:

   - ``profile_clientssl_stat`` and ``profile_serverssl_stat`` are in the BNK 2.4.0 Observer
     descriptor. If not, the Observer logs a table error (see TC-05 Scenario 5).
   - ``tls_version`` and ``tls_cipher`` are exposed by those tables. The BNPP metrics table
     derives them from ``ssl_protocol_stat`` and ``ssl_cipher_stat``; if the descriptor maps
     those separately, add a job for each.
   - ``aggregation: Aggregated`` keeps the ``tmm`` label that the verification checks for.
     TC-06 makes the same assumption.

   Steps 3 and 4 still say "Apply ScrapeClass, ScrapeTemplate and Scrape" and "Apply
   ObserverConfig"; apply the files below instead.

Custom resources
----------------

Each file holds one resource, numbered in the order listed here.

.. list-table::
   :header-rows: 1
   :widths: 40 15 25 20

   * - File
     - Kind
     - Name
     - Namespace
   * - ``01-scrapeclass-f5-ssl-metrics-class.yaml``
     - ScrapeClass
     - ``f5-ssl-metrics-class``
     - (cluster)
   * - ``02-scrapetemplate-f5-ssl-metrics-template.yaml``
     - ScrapeTemplate
     - ``f5-ssl-metrics-template``
     - f5-alpha

ScrapeClass ``f5-ssl-metrics-class``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Replaces the test plan's YAML; see *Deviation from the test plan*.

.. literalinclude:: 01-scrapeclass-f5-ssl-metrics-class.yaml
   :language: yaml

ScrapeTemplate ``f5-ssl-metrics-template``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Replaces the test plan's YAML; see *Deviation from the test plan*.

.. literalinclude:: 02-scrapetemplate-f5-ssl-metrics-template.yaml
   :language: yaml
