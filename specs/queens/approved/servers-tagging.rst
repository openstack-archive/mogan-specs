..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============
Server tagging
==============

https://blueprints.launchpad.net/mogan/+spec/server-tags-support

This aims to add support for tagging servers.

Problem description
===================

Mogan should have tags field for every server, which can be used to
divide the servers into some groups. Then we can do list by tags to
get a group of servers with same properties like managed servers.

Use Cases
---------

* As a user, I want to classify bare metal servers based on some properties
  such as os type, server function.


Proposed change
===============

* Add APIs that allow a user to add, remove, and list tags for a server.

* Add tag filter parameters to server list API to allow searching for servers
  based on one or more string tags.

Alternatives
------------

None

Data model impact
-----------------

New `mogan.objects.tag.ServerTag` and `mogan.objects.tag.ServerTagList` object
would be added to the object model.

The `mogan.objects.tag.ServerTagList` field in the python object model
will be populated on-demand (i.e. not eager-loaded).

A tag should be defined as a Unicode string no longer than 255 characters
in length, with an index on this field.

Tags are strings attached to an entity with the purpose of classification
into groups. To simplify requests that specify lists of tags, the comma
character is not allowed to be in a tag name.

The proposed change will be adding the following fields to the ServerTag
object with their data type and default value for migrations.

+-----------------------+--------------+
| Field Name            | Field Type   |
+=======================+==============+
| server_id             | Integer      |
+-----------------------+--------------+
| tag                   | String       |
+-----------------------+--------------+

For the database schema, the following table constructs would suffice ::

    op.create_table(
        'server_tags',
        sa.Column('created_at', sa.DateTime(), nullable=True),
        sa.Column('updated_at', sa.DateTime(), nullable=True),
        sa.Column('server_id', sa.Integer(), nullable=False),
        sa.Column('tag', sa.String(length=255), nullable=False),
        sa.PrimaryKeyConstraint('server_id', 'tag'),
        sa.ForeignKeyConstraint(['server_id'],
                                ['servers.id']),
        sa.Index('server_tags_tag_idx', 'tag'),
        mysql_ENGINE='InnoDB',
        mysql_DEFAULT_CHARSET='UTF8'
    )

REST API impact
---------------

We will follow the `API Working Group's specification for tagging`_, rather
than invent our own.

.. _API Working Group's specification for tagging: http://specs.openstack.org/openstack/api-wg/guidelines/tags.html

Will support addressing individual tags.


RPC API impact
--------------

None

State Machine Impact
--------------------

None

Client (CLI) impact
-------------------

Add tags CRUD operations commands:

* openstack baremetalcompute server tag list <server_uuid>
* openstack baremetalcompute server tag update <server_uuid> <op> <tags>

<op> Operation: 'add' or 'remove'

For individual tag:
* openstack baremetalcompute server tag add <server_uuid> <tag>
* openstack baremetalcompute server tag remove <server_uuid> <tag>

Add tag-list filtering support to node-list command:

* openstack baremetalcompute server list --tag tag1 --tag tag2
* openstack baremetalcompute server list --tag-any tag1 --tag-any tag2
* openstack baremetalcompute server list --not-tag tag3

Multiple --tag will be used to filter results in an AND expression, and
--tag-any for OR expression, allowing for exclusionary tags via the
--not-tag option.

Driver API impact
-----------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Scalability impact
------------------

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
  Tao Li <litao3721@126.com>

Work Items
----------

* Update api-ref document to describe tag information.
* Add `server_tags` table with a migration.
* Add DB API layer for CRUD operations on server tags.
* Added DB API layer for server tag list filtering support.
* Add ServerTag, ServerTagList objects and a new tags field to Server object.
* Add REST API for CRUD operations on server tags.
* Add REST API for server tag list filtering support.
* Add tags support for creating servers.
* python-moganclient additions and modifications.


Dependencies
============

None


Testing
=======

Add unit tests.
Add tempest API tests.


Documentation Impact
====================

Mogan API and python-moganclient will need to be updated to accompany
this change.


References
==========

1. http://specs.openstack.org/openstack/api-wg/guidelines/tags.html
