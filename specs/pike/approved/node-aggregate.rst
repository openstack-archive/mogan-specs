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

* An enterprise admin wants to divide baremetal nodes into logical groups
which can be linked to flavors.


Proposed change
===============

A new mogan.objects.aggregate.Aggregate object would be added to the object
model. Then, add APIs for only admins that allows to add, remove, and list
aggregates, also admins should be able to add nodes to the specific aggregate
and remove nodes from it.

A new filter should be added to handle flavor extra spec key/value matching
with aggregate metadata.

Alternatives
------------

Leverage Placement aggregates to achieve this.

Data model impact
-----------------

The proposed change will be add the following fields to the aggregate object
with their data type and default value for migrations.

+-----------------------+--------------+
| Field Name            | Field Type   |
+=======================+==============+
|          id           | Integer      |
+-----------------------+--------------+
|         uuid          | UUID         |
+-----------------------+--------------+
|         name          | String       |
+-----------------------+--------------+
|         nodes         | ListOfStrings|
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
        'aggregate_nodes',
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('node', sa.String(length=36), nullable=False),
        sa.Column('aggregate_id', sa.Integer(), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.ForeignKeyConstraint(['aggregate_id'],
                                ['aggregates.id']),
        mysql_ENGINE='InnoDB',
        mysql_DEFAULT_CHARSET='UTF8'
    )

    op.create_table(
        'aggregate_metadta',
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('key', sa.String(length=255), nullable=False),
        sa.Column('value', sa.String(length=255), nullable=False),
        sa.Column('aggregate_id', sa.Integer(), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.ForeignKeyConstraint(['aggregate_id'],
                                ['aggregates.id']),
        mysql_ENGINE='InnoDB',
        mysql_DEFAULT_CHARSET='UTF8'
    )

REST API impact
---------------

- To create a new aggregate, a user will ::

    POST /v1/aggregates

  With a body containing the JSON description of the aggregate.

- To list aggregates, a user will ::

    GET /v1/aggregates

- To delete a aggregate, a user will ::

    DELETE /v1/aggregates/{aggregates_uuid}

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

* Add aggregates, aggregate_nodes, aggregate_metadata tables.
* Add aggregate object.
* Add APIs that allows an admin to add, remove, and list node aggregates.
* Add APIs that allows and admin to add/remove nodes to a aggregate.
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
