..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============
Node Aggregates
===============

https://blueprints.launchpad.net/mogan/+spec/node-aggregate

This introduces the concept of aggregate into Mogan, which is quite like Nova
host aggregate, but we are based on baremetal nodes. Node aggregate allows the
partition of baremetal nodes into logical groups for server distribution.


Problem description
===================

Currently Mogan only have a concept of availability zones which is for
providing isolation and redundancy from other availability zones. We need a
mechanism to further partitioning baremetal nodes.

Use Cases
---------

* As a cloud operator, I want to classify baremetal nodes based on the
  location like rack, row, cage, zone, DC.

* As a cloud operator, I want to classify baremetal nodes based on the
  hardware specs like with GPU, FPGA, ceph storage backend.


Proposed change
===============

* A new mogan.objects.aggregate.Aggregate object would be added to the object
model.

* Add a set of API that only allow admins to create, delete, and list
aggregates, also admins should be able to add nodes to the specific aggregate
and remove nodes from it.

* When do resources update we will cache the resource providers and the map of
aggregates with resource providers. Add a node list API for admins which will
derive resource providers from the cache.

* Before scheduling, mogan will handle the flavor resource_traits matching with
aggregate metadata, then got a list of aggregates, and pass it to placement
with `member_of` parameter when listing resource providers::

    /resource_providers?member_of=in:{agg1_uuid},{agg2_uuid},{agg3_uuid}

Alternatives
------------

Implement aggregates on mogan side instead of leveraging placement aggregates.

Data model impact
-----------------

The proposed change will be adding the following fields to the aggregate
object with their data type and default value for migrations.

+-----------------------+--------------+
| Field Name            | Field Type   |
+=======================+==============+
|          id           | Integer      |
+-----------------------+--------------+
|         uuid          | UUID         |
+-----------------------+--------------+
|         name          | String       |
+-----------------------+--------------+
|        metadata       | DictOfStrings|
+-----------------------+--------------+

For the database schema, the following table constructs would suffice ::

    op.create_table(
        'aggregates',
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('uuid', sa.String(length=36), nullable=False),
        sa.Column('name', sa.String(length=255), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        mysql_ENGINE='InnoDB',
        mysql_DEFAULT_CHARSET='UTF8'
    )

    op.create_table(
        'aggregate_metadata',
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.Column('id', sa.Integer(), primary_key=True, nullable=False),
        sa.Column('key', sa.String(length=255), nullable=False),
        sa.Column('value', sa.String(length=255), nullable=False),
        sa.Column('aggregate_id', sa.Integer(), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.ForeignKeyConstraint(['aggregate_id'],
                                ['aggregate.id']),
        mysql_ENGINE='InnoDB',
        mysql_DEFAULT_CHARSET='UTF8'
    )

REST API impact
---------------

- To create a new aggregate, a user will ::

    POST /v1/aggregates

  With a body containing the JSON description of the aggregate.

  JSON Schema::

    {
        "type": "object",
        "properties": {
            "name": {"type": "string", "minLength": 1, "maxLength": 255},
            "metadata": {
                'type': 'object',
                'patternProperties': {
                    '^[a-zA-Z0-9-_:. ]{1,255}$': {
                        'type': 'string', 'maxLength': 255
                    }
                },
                'additionalProperties': False
            },
        },
        "required": ["name"],
        "additionalProperties": False,
    }

- To list aggregates, a user will ::

    GET /v1/aggregates

- To show aggregate details, a user will ::

    GET /v1/aggregates/{aggregate_id}

- To update aggregate, a user will ::

    PATCH /v1/aggregates/{aggregate_id}

  With a body containing the JSON description of the fileds to be updated.

  Example Update Aggregate: JSON request::

  [
      {
          "op": "replace",
          "path": "/name",
          "value": "foo"
      },
      {
          "op": "add",
          "path": "/metadata/k1",
          "value": "v1"
      }
  ]

- To delete an aggregate, a user will ::

    DELETE /v1/aggregates/{aggregate_id}

- To add nodes to an aggregate, a user will ::

    POST /v1/aggregates/{aggregate_id}/nodes

  With a body containing a list of node uuid to be added to the aggregate.

- To remove node from an aggregate, a user will ::

    DELETE /v1/aggregates/{aggregate_id}/nodes/{node}

- To get nodes from an aggregate, a user will ::

    GET /v1/aggregates/{aggregate_id}/nodes

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <niu-zglinux>

Work Items
----------

* Add aggregate object.
* Add APIs that allows an admin to add, remove, and list node aggregates.
* Add APIs that allows an admin to add/remove nodes to an aggregate.
* Add new CLIs to manage node aggregates.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about node aggregates will be added.

References
==========

None
