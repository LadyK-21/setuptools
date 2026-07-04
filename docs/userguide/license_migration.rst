.. _license-migration:

-------------------------------------------------------
Migrating to PEP 639 License Expressions
-------------------------------------------------------

Starting with setuptools v77.0.0, the ``project.license`` field now supports
`SPDX license expressions <https://spdx.org/licenses/>`_ as defined in
:pep:`639`. The previous TOML-table-based format (``license = {text = "..."}``)
is deprecated and will be removed in a future version.

This guide helps you migrate your project's license configuration.

.. contents:: Table of Contents
   :local:
   :depth: 2

Background
----------

The ``project.license`` field in ``pyproject.toml`` historically accepted a
TOML table with a ``text`` key:

.. code-block:: toml

   # Old format (deprecated)
   license = {text = "Apache-2.0"}

Starting with setuptools v77.0.0, you can use a simple SPDX license expression:

.. code-block:: toml

   # New format (PEP 639)
   license = "Apache-2.0"

The SPDX expression format is more standardized and allows for complex license
combinations (e.g., ``"Apache-2.0 AND MIT"``).

Migration Strategies
--------------------

Strategy 1: Target Python 3.9+ Only
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your project only supports Python 3.9 or later, you can use setuptools v77+
exclusively:

.. code-block:: toml

   [build-system]
   requires = ["setuptools>=77"]
   build-backend = "setuptools.build_meta"

   [project]
   name = "my_package"
   requires-python = ">=3.9"
   license = "Apache-2.0"  # PEP 639 SPDX expression

This is the simplest approach and recommended for new projects.

Strategy 2: Support Python 3.8 and Later
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you need to support Python 3.8, you have two options:

**Option A: Use the old format (recommended for now)**

Keep using the TOML-table format until Python 3.8 reaches end-of-life
(October 2024):

.. code-block:: toml

   [build-system]
   requires = ["setuptools"]
   build-backend = "setuptools.build_meta"

   [project]
   name = "my_package"
   requires-python = ">=3.8"
   license = {text = "Apache-2.0"}  # TOML-table format (deprecated but works)

**Option B: Conditional build system requirements**

Use environment markers to conditionally require setuptools v77+:

.. code-block:: toml

   [build-system]
   requires = [
       "setuptools>=77; python_version>='3.9'",
       "setuptools; python_version<'3.9'",
   ]
   build-backend = "setuptools.build_meta"

   [project]
   name = "my_package"
   requires-python = ">=3.8"
   license = "Apache-2.0"  # PEP 639 SPDX expression

.. note::
   This approach may not work with all build frontends. Test thoroughly
   before adopting.

Strategy 3: Use License-Files Instead
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your license is a standard file (e.g., ``LICENSE``, ``LICENSE.txt``), you
can use the ``license-files`` field instead:

.. code-block:: toml

   [build-system]
   requires = ["setuptools"]
   build-backend = "setuptools.build_meta"

   [project]
   name = "my_package"
   requires-python = ">=3.8"
   license-files = ["LICENSE"]  # Point to the license file

This avoids the deprecation warning entirely and works with all setuptools
versions that support ``pyproject.toml``.

Common SPDX License Expressions
-------------------------------

Here are some common SPDX expressions:

- ``"MIT"`` - MIT License
- ``"Apache-2.0"`` - Apache License 2.0
- ``"BSD-3-Clause"`` - BSD 3-Clause License
- ``"GPL-3.0-only"`` - GNU General Public License v3.0 only
- ``"Apache-2.0 OR MIT"`` - Dual license (either Apache-2.0 or MIT)
- ``"Apache-2.0 AND MIT"`` - Both licenses apply

For the full list, see the `SPDX License List <https://spdx.org/licenses/>`_.

Troubleshooting
---------------

**"SetuptoolsDeprecationWarning: `project.license` as a TOML table is deprecated"**

This warning appears when using the old TOML-table format with setuptools v77+.
To suppress it:

1. Follow one of the migration strategies above, or
2. Add a ``filterwarnings`` in your test configuration if you can't migrate yet

**"Error: Invalid SPDX expression"**

Ensure your license expression uses valid SPDX identifiers. Common mistakes:

- Using ``"GPL-3"`` instead of ``"GPL-3.0-only"``
- Using ``"BSD"`` instead of ``"BSD-3-Clause"``
- Forgetting quotes around the expression

**Build fails on Python 3.8**

If you're using setuptools v77+ in ``requires`` without a Python version
marker, the build will fail on Python 3.8. Use conditional requires as
shown in Strategy 2.

References
----------

- :pep:`639` - License Expression in Project Metadata
- `SPDX License List <https://spdx.org/licenses/>`_
- `Writing your pyproject.toml <https://packaging.python.org/en/latest/guides/writing-pyproject-toml/>`_
- `Setuptools Changelog <https://setuptools.pypa.io/en/latest/history.html>`_
