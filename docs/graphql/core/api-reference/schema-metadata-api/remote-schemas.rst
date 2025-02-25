.. meta::
   :description: Manage remote schemas with the Hasura schema/metadata API
   :keywords: hasura, docs, schema/metadata API, API reference, remote schema

.. _api_remote_schemas:

Schema/Metadata API Reference: Remote schemas
=============================================

.. contents:: Table of contents
  :backlinks: none
  :depth: 1
  :local:

Introduction
------------

Add/Remove a remote GraphQL server as remote schema in Hasura GraphQL engine.

.. admonition:: Deprecation

  In versions ``v2.0.0`` and above, the schema/metadata API is deprecated in favour of the :ref:`schema API <schema_apis>` and the
  :ref:`metadata API <metadata_apis>`.

  Though for backwards compatibility, the schema/metadata APIs will continue to function.

.. _add_remote_schema:

add_remote_schema
-----------------

``add_remote_schema`` is used to add a remote GraphQL server as remote schema. GraphQL engine stitches it's schema with existing.

An example request as follows:

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   X-Hasura-Role: admin

   {
       "type": "add_remote_schema",
       "args": {
           "name": "my remote schema",
           "definition": {
               "url": "https://remote-server.com/graphql",
               "headers": [{"name": "X-Server-Request-From", "value": "Hasura"}],
               "forward_client_headers": false,
               "timeout_seconds": 60
           },
           "comment": "some optional comment"
       }
   }


.. _add_remote_schema_syntax:

.. list-table::
   :header-rows: 1

   * - Key
     - Required
     - Schema
     - Description
   * - name
     - true
     - :ref:`RemoteSchemaName`
     - Name of the remote schema
   * - definition
     - true
     - :ref:`RemoteSchemaDef`
     - Definition for the remote schema
   * - comment
     - false
     - Text
     - comment

.. _update_remote_schema:

update_remote_schema
-----------------

``update_remote_schema`` is used to update the configuration of a remote schema. If the remote schema URL has changed 
then it will perform a introspection as well. After introspection, if there are any inconsistencies detected with other 
metadata objects (like remote relationships or remote schema permissions) they will be reported as `inconsistent_metadata`.

An example request as follows:

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   X-Hasura-Role: admin

   {
       "type": "update_remote_schema",
       "args": {
           "name": "my remote schema",
           "definition": {
               "url": "https://remote-server.com/graphql",
               "headers": [{"name": "X-Server-Request-From", "value": "Hasura"}],
               "forward_client_headers": false,
               "timeout_seconds": 60
           },
           "comment": "some optional comment"
       }
   }


.. _update_remote_schema_syntax:

.. list-table::
   :header-rows: 1

   * - Key
     - Required
     - Schema
     - Description
   * - name
     - true
     - :ref:`RemoteSchemaName`
     - Name of the remote schema
   * - definition
     - true
     - :ref:`RemoteSchemaDef`
     - Definition for the remote schema
   * - comment
     - false
     - Text
     - comment

.. _remove_remote_schema:

remove_remote_schema
--------------------

``remove_remote_schema`` is used to delete a remote schema. GraphQL engine de-stitches it's schema.

An example request as follows:

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   X-Hasura-Role: admin

   {
       "type": "remove_remote_schema",
       "args": {
           "name": "my remote schema"
       }
   }

.. _remove_remote_schema_syntax:

.. list-table::
   :header-rows: 1

   * - Key
     - Required
     - Schema
     - Description
   * - name
     - true
     - :ref:`RemoteSchemaName`
     - Name of the remote schema

.. _reload_remote_schema:

reload_remote_schema
--------------------

``reload_remote_schema`` is used to refresh schema of the remote server. GraphQL engine refetches schema from server and stitches.

An example request as follows:

.. code-block:: http

   POST /v1/query HTTP/1.1
   Content-Type: application/json
   X-Hasura-Role: admin

   {
       "type": "reload_remote_schema",
       "args": {
           "name": "my remote schema"
       }
   }

.. _reload_remote_schema_syntax:

.. list-table::
   :header-rows: 1

   * - Key
     - Required
     - Schema
     - Description
   * - name
     - true
     - :ref:`RemoteSchemaName`
     - Name of the remote schema
