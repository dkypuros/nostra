Overview
===============

Example RST text.


.. _Section1:

Weaviate - Vector Database
============================

Base Installation of Weaviate - Local/Fedora
----------------------------------------------

Pull image
+++++++++++++

.. code-block:: bash

    podman pull docker.io/semitechnologies/weaviate:latest

.. code-block:: bash

    podman image ls

Setup volume (storage)
++++++++++++++++++++++

.. code-block:: bash

    podman volume create weaviate-db-store

.. code-block:: bash

    podman volume ls

Setup Yaml for compose
+++++++++++++++++++++++++++

.. tip::

    Create this file in the same directory where they'll be running the podman-compose up -d command

:code:`docker-compose.yaml`

.. code-block:: yaml
   :linenos:

    version: '3'
    services:
    weaviate-node-1:
        init: true
        command:
        - --host
        - 0.0.0.0
        - --port
        - '8080'
        - --scheme
        - http
        image: docker.io/semitechnologies/weaviate
        ports:
        - 8080:8080
        - 6060:6060
        restart: on-failure:0
        volumes:
        - weaviate-db-store:/var/lib/weaviate
        environment:
        LOG_LEVEL: 'debug'
        QUERY_DEFAULTS_LIMIT: 25
        AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
        PERSISTENCE_DATA_PATH: '/var/lib/weaviate'
        DEFAULT_VECTORIZER_MODULE: 'none'
        CLUSTER_HOSTNAME: 'node1'
        CLUSTER_GOSSIP_BIND_PORT: '7100'
        CLUSTER_DATA_BIND_PORT: '7101'
    volumes:
    weaviate-db-store:


Build the container
++++++++++++++++++++++++++

.. code-block:: bash

    podman-compose up -d && podman-compose logs -f weaviate

That's it for the base installation of Weaviate.


Adding Modules
----------------------------------------------------------

Find the correct image
++++++++++++++++++++++++++

.. code-block:: bash

    podman search docker.io/semitechnologies/

Let's Add  "Text2Transformers" & "Weaviate Playground"
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. warning::

    This thing is 4.4GiB

.. code-block:: bash

    podman pull docker.io/semitechnologies/transformers-inference:sentence-transformers-multi-qa-MiniLM-L6-cos-v1

:code:`docker-compose.yaml`

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 4, 26, 14, 8, 18

    version: '3'
    services:
    weaviate:
        image: docker.io/semitechnologies/weaviate
        ports:
        - "8080:8080"
        volumes:
        - weaviate-db-store:/var/lib/weaviate
        environment:
        QUERY_DEFAULTS_LIMIT: 20
        AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
        PERSISTENCE_DATA_PATH: "/var/lib/weaviate"
        DEFAULT_VECTORIZER_MODULE: text2vec-transformers
        ENABLE_MODULES: text2vec-transformers
        TRANSFORMERS_INFERENCE_API: http://t2v-transformers:8080
        CLUSTER_HOSTNAME: 'node1'
    t2v-transformers:
        image: docker.io/semitechnologies/transformers-inference:sentence-transformers-multi-qa-MiniLM-L6-cos-v1
        environment:
        ENABLE_CUDA: '0'  # set to '1' to enable CUDA if running with NVIDIA GPUs
    weaviate-playground:
        image: docker.io/semitechnologies/weaviate-playground
        ports:
        - "8081:80"
    volumes:
    weaviate-db-store:



.. code-block:: bash

    podman-compose up -d && \
    podman-compose logs -f weaviate


View the Playground
++++++++++++++++++++++++

The Service:

.. code-block:: bash

    http://localhost:8080/v1

The playground aka GUI:

Paste the "service URL" into the "connect to weavate" link and add "graphql" per the instructions. :code:`http://localhost:8080/v1/graphql`

.. code-block:: bash

    http://localhost:8081/


The Playground / GUI is up

.. image:: _static/images/screen-shot-01.png