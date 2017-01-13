============================================
OpenStack Baremetal Computing Specifications
============================================

This git repository is used to hold approved design specifications for
additions to the Baremetal Provisioning program, and in particular, the Mogan
project.  Reviews of the specs are done in gerrit, using a similar workflow to
how we review and merge changes to the code itself.

The layout of this repository is::

  specs/<release>/

Specifications must follow the template which can be found at
`specs/template.rst`.

blueprints::

  http://blueprints.launchpad.net/mogan

For more information about working with gerrit, see::

  http://docs.openstack.org/infra/manual/developers.html#development-workflow

To validate that the specification is syntactically correct (i.e. get more
confidence in the Jenkins result), please execute the following command::

  $ tox

After running ``tox``, the documentation will be available for viewing in HTML
format in the ``doc/build/`` directory.

To quickly preview just the rst syntax rendering (without Sphinx features)
of a single spec file execute the following command::

    $ tox -evenv rst2html.py <path-to-your-spec.rst> <path-to-output.html>

and open ``<path-to-output.html>`` in your browser.
Running full ``tox`` is still advised before submitting your patch.
