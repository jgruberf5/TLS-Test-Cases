.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-06: Validate that HTTP profile metrics are collected from BIG-IP TMM instances and exported to Prometheus via OTEL Collector (number_reqs, get_reqs, post_reqs, resp_2xx_cnt, resp_5xx_cnt, number_reqs)
===========================================================================================================================================================================================================

:Test case: 6
:Priority: P1
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

- Observer configured with multiple collection jobs (HTTP, TCP, VS, Pool)
- BIG-IP Virtual Server with HTTP/TCP traffic
- Fluentd forwarding to Kafka
- OTEL Collector forwarding to Prometheus

Steps
-----

1. Deploy HTTP Virtual Server with traffic management policy
2. Apply ScrapeClass, ScrapeTemplate and Scrape with profile_http_stat table definition
3. Apply ScrapeTemplate with http-profile-stats collection job (interval: 30s)
4. Wait 60 seconds for TMM to populate profile_http_stat table
5. Generate HTTP traffic across both TMM instances:

   - Send 5 HTTP GET requests to Virtual Server (distributed across TMM-0 and TMM-1)
   - Send 3 HTTP POST requests
   - Send 2 HTTP PUT requests
   - Total: 10 HTTP requests

6. Wait 90 seconds (3 collection intervals)
7. Query Prometheus for HTTP metrics (bnpp_http_*)
8. Verify metrics are aggregated across both TMM instances

Verification
------------

- Verify Prometheus shows HTTP metrics with labels: ``{tmm="0"}`` and ``{tmm="1"}``
- Verify Metrics present from **tmm_profile_http_stat** table:

  - ``number_reqs`` ≥ 10 (total requests across TMMs)
  - ``get_reqs`` ≥ 5 (GET requests)
  - ``post_reqs`` ≥ 3 (POST requests)
  - ``resp_2xx_cnt`` > 0 (HTTP 2xx responses)
  - ``resp_5xx_cnt`` ≥ 0 (HTTP 5xx responses)

- Verify Data aggregated correctly: ``sum(number_reqs)`` = total requests
- Verify Timestamp freshness: Latest scrape within last 60 seconds
- Verify No missing TMM instances in labels

Custom resources
----------------

Each file holds one resource, numbered in the order the test plan declares them.

.. list-table::
   :header-rows: 1
   :widths: 40 15 25 20

   * - File
     - Kind
     - Name
     - Namespace
   * - ``01-scrapeclass-f5-http-metrics-class.yaml``
     - ScrapeClass
     - ``f5-http-metrics-class``
     - (cluster)
   * - ``02-scrapetemplate-f5-http-metrics-template.yaml``
     - ScrapeTemplate
     - ``f5-http-metrics-template``
     - f5-alpha
   * - ``03-scrape-f5-http-metrics-scrape.yaml``
     - Scrape
     - ``f5-http-metrics-scrape``
     - f5-alpha

ScrapeClass ``f5-http-metrics-class``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *(untitled section)*.

.. literalinclude:: 01-scrapeclass-f5-http-metrics-class.yaml
   :language: yaml

ScrapeTemplate ``f5-http-metrics-template``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *(untitled section)*.

.. literalinclude:: 02-scrapetemplate-f5-http-metrics-template.yaml
   :language: yaml

Scrape ``f5-http-metrics-scrape``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *(untitled section)*.

.. literalinclude:: 03-scrape-f5-http-metrics-scrape.yaml
   :language: yaml
