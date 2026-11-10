.. Generated from Confluence page https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform (version 35).
.. Regenerate rather than hand-edit if the source page changes.

TC-01: Verify FLO-managed Fluentd can forward logs to Kafka using custom ConfigMap and Secret on IBM ROK Platform
=================================================================================================================

:Test case: 1
:Priority: P0
:Target release: BNK 2.4.0
:Platform: BNK for BNPP - Red Hat OpenShift on IBM Cloud
:Source: `Confluence test plan (v35) <https://docs.f5net.com/spaces/PD/pages/1262164643/BNPP+-+Test+Plan+for+HTTP+TLS+metrics+Gateway+API+And+log+export+for+IBM+ROK+Platform>`__

Pre-conditions
--------------

- Kafka cluster deployed and operational with 3 broker
- Kafka brokers accessible via mTLS on port 9094
- Prometheus server deployed and accessible on port 9090
- OTEL Collector deployed in f5-common namespace
- Observer deployed in f5-common namespace
- Fluentd/Fluent Bit deployed in logging namespace
- Network connectivity verified from K8s to Kafka on port 9094
- Network connectivity verified from K8s to Prometheus on port 9090
- Kafka topic bnpp-http-logs created and verified
- Kafka topic bnpp-https-logs created and verified
- mTLS certificates provisioned in toda-kafka-client-tls secret
- Evertrust Kafka Secret created & mounted
- cert-manager internal secret exists
- Both volumes mounted in Fluentd pod
- FLO must be updated to support mounting externally-issued certificates into the Fluentd pod:
- New field in CNEInstance CR (or Fluentd component spec): 'externalCertificates' — a list of Secret references to mount into the Fluentd pod
- Volume mount propagation: FLO's Fluentd handler must propagate the external certificate volume mounts to the Fluentd Deployment spec
- Pattern: Similar to existing otel/stats certificate handling where external secrets are mounted as volumes

Steps
-----

- Create Kafka Certificates (using Cert-Manager)
- Copy Kafka Certificates to FLO Custom Secret
- Add Kafka Output Configuration to FLO Custom ConfigMap
- Restart FLO-Managed Fluentd
- Verify Kafka Connection and Log Forwarding

Verification
------------

- Verify Certificates successfully copied to f5-toda-fluentd-custom-secret
- Verify Kafka configuration added to f5-toda-fluentd-custom ConfigMap
- Verify FLO reconciles and applies custom configuration (does not reset it)
- Verify Fluentd pods restart and mount custom secret/configmap
- Verify Fluentd connects to Kafka via mTLS (port 9094)
- Verify Test logs appear in Kafka topic within 30 seconds
- Verify No authentication or connection errors in Fluentd logs
- Verify **Custom configuration persists across Fluentd restarts**

Secret Validation
-----------------

.. code-block:: bash

   # Verify Evertrust Kafka secret
   kubectl get secret evertrust-kafka-secret -n logging \
     -o jsonpath='{.data}' | jq 'keys'
   # Verify cert-manager internal secret
   kubectl get secret fluentd-internal-tls -n logging \
     -o jsonpath='{.data}' | jq 'keys'
   # Verify both volume mounts inside Fluentd pod
   kubectl exec -it <fluentd-pod> -n logging -- ls /certs/internal/
   kubectl exec -it <fluentd-pod> -n logging -- ls /certs/kafka/

Custom resources
----------------

Each file holds one resource, numbered in the order the test plan declares them.

.. note::

   ``02-certificate-kafka-server-cert.yaml`` targets namespace ``kafka``, which ``03-namespace-kafka.yaml`` creates. Apply ``03-namespace-kafka.yaml`` first.

.. list-table::
   :header-rows: 1
   :widths: 40 15 25 20

   * - File
     - Kind
     - Name
     - Namespace
   * - ``01-clusterissuer-vault-acme-issuer.yaml``
     - ClusterIssuer
     - ``vault-acme-issuer``
     - (cluster)
   * - ``02-certificate-kafka-server-cert.yaml``
     - Certificate
     - ``kafka-server-cert``
     - kafka
   * - ``03-namespace-kafka.yaml``
     - Namespace
     - ``kafka``
     - (cluster)
   * - ``04-service-kafka-cluster-kafka-bootstrap.yaml``
     - Service
     - ``kafka-cluster-kafka-bootstrap``
     - kafka
   * - ``05-deployment-kafka-local.yaml``
     - Deployment
     - ``kafka-local``
     - kafka
   * - ``06-kafkatopic-bnpp-http-logs.yaml``
     - KafkaTopic
     - ``bnpp-http-logs``
     - kafka
   * - ``07-kafkatopic-bnpp-https-logs.yaml``
     - KafkaTopic
     - ``bnpp-https-logs``
     - kafka
   * - ``08-kafkatopic-kubernetes-logs.yaml``
     - KafkaTopic
     - ``kubernetes-logs``
     - kafka
   * - ``09-kafkatopic-http-logs.yaml``
     - KafkaTopic
     - ``http-logs``
     - kafka
   * - ``10-clusterflow-http-logs-to-kafka.yaml``
     - ClusterFlow
     - ``http-logs-to-kafka``
     - logging
   * - ``11-clusteroutput-fluentd-to-kafka-output.yaml``
     - ClusterOutput
     - ``fluentd-to-kafka-output``
     - logging
   * - ``12-clusteroutput-bnpp-http-to-kafka.yaml``
     - ClusterOutput
     - ``bnpp-http-to-kafka``
     - logging
   * - ``13-clusteroutput-bnpp-https-to-kafka.yaml``
     - ClusterOutput
     - ``bnpp-https-to-kafka``
     - logging
   * - ``14-clusterflow-bnpp-http-flow.yaml``
     - ClusterFlow
     - ``bnpp-http-flow``
     - logging
   * - ``15-clusterflow-bnpp-https-flow.yaml``
     - ClusterFlow
     - ``bnpp-https-flow``
     - logging

ClusterIssuer ``vault-acme-issuer``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *ClusterIssuer CR*.

.. literalinclude:: 01-clusterissuer-vault-acme-issuer.yaml
   :language: yaml

Certificate ``kafka-server-cert``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Kafka mtls Certificate CR*.

.. literalinclude:: 02-certificate-kafka-server-cert.yaml
   :language: yaml

Namespace ``kafka``
~~~~~~~~~~~~~~~~~~~

From the test plan section *kafka deployment*.

.. literalinclude:: 03-namespace-kafka.yaml
   :language: yaml

Service ``kafka-cluster-kafka-bootstrap``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *kafka deployment*.

.. literalinclude:: 04-service-kafka-cluster-kafka-bootstrap.yaml
   :language: yaml

Deployment ``kafka-local``
~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *kafka deployment*.

.. literalinclude:: 05-deployment-kafka-local.yaml
   :language: yaml

KafkaTopic ``bnpp-http-logs``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Kafka Topic CR*.

.. literalinclude:: 06-kafkatopic-bnpp-http-logs.yaml
   :language: yaml

KafkaTopic ``bnpp-https-logs``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Kafka Topic CR*.

.. literalinclude:: 07-kafkatopic-bnpp-https-logs.yaml
   :language: yaml

KafkaTopic ``kubernetes-logs``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Kafka Topic CR*.

.. literalinclude:: 08-kafkatopic-kubernetes-logs.yaml
   :language: yaml

KafkaTopic ``http-logs``
~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Kafka Topic CR*.

.. literalinclude:: 09-kafkatopic-http-logs.yaml
   :language: yaml

ClusterFlow ``http-logs-to-kafka``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Clusterflow CR*.

.. literalinclude:: 10-clusterflow-http-logs-to-kafka.yaml
   :language: yaml

ClusterOutput ``fluentd-to-kafka-output``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *Clusteroutput CR*.

.. literalinclude:: 11-clusteroutput-fluentd-to-kafka-output.yaml
   :language: yaml

ClusterOutput ``bnpp-http-to-kafka``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *ClusterOutput and ClusterFlow CR*.

.. literalinclude:: 12-clusteroutput-bnpp-http-to-kafka.yaml
   :language: yaml

ClusterOutput ``bnpp-https-to-kafka``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *ClusterOutput and ClusterFlow CR*.

.. literalinclude:: 13-clusteroutput-bnpp-https-to-kafka.yaml
   :language: yaml

ClusterFlow ``bnpp-http-flow``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *ClusterOutput and ClusterFlow CR*.

.. literalinclude:: 14-clusterflow-bnpp-http-flow.yaml
   :language: yaml

ClusterFlow ``bnpp-https-flow``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the test plan section *ClusterOutput and ClusterFlow CR*.

.. literalinclude:: 15-clusterflow-bnpp-https-flow.yaml
   :language: yaml
