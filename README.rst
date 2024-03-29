This repo was archived on November 8, 2022. 

httplib2shim
============

|Build Status| |Coverage Status| |PyPI Version|

``httplib2shim`` is a wrapper over ``httplib2`` that uses ``urllib3`` to perform HTTP requests. This library is intended to help existing legacy libraries (and their users) to migrate away from ``httplib2``. It is not intended to be a general purpose replacement for ``httplib2``. It does not support every feature and edge case for ``httplib2``, although contributions are welcome in order to help us cover these cases.

Presently, ``httplib2shim`` passes the test suite for ``httplib2``. A few non-applicable tests were disabled, and it's very possible that the tests do not account for behavior that is depended on by clients.


Usage
-----

It's recommended to install ``urllib3[secure]`` before installing ``httplib2shim``:

.. code:: bash

    pip install urllib3[secure] httplib2shim

Usage is straightforward. You can substitute ``httplib2shim.Http`` anywhere ``httplib2.Http`` is used. E.g. for the `oauth2client <https://github.com/google/oauth2client>`_ use:

.. code:: python

    import httplib2shim
    from oauth2client.client import GoogleCredentials


    credentials = GoogleCredentials.get_application_default()
    http = httplib2shim.Http()
    credentials.authorize(http)

    # http is now authorized with OAuth2 credentials and uses urllib3 under
    # the covers.

and for the `google-auth <https://github.com/GoogleCloudPlatform/google-auth-library-python>`_ use:

.. code:: python

    import httplib2shim
    import google.auth
    import google_auth_httplib2

    credentials, _ = google.auth.default()
    http = httplib2shim.Http()
    http = google_auth_httplib2.AuthorizedHttp(credentials, http=http)

Alternatively, if you do not control the construction of the ``Http`` object, you can use ``httplib2shim.patch()`` to monkey-patch the ``httplib2.Http`` class to point to ``httplib2shim.Http()``:

.. code:: python

    import httplib2shim
    httplib2shim.patch()

    from googleapiclient.discovery import build
    from oauth2client.client import GoogleCredentials

    credentials = GoogleCredentials.get_application_default()

    # build constructs its own httplib2.Http instance.
    service = build('compute', 'v1', credentials=credentials)

    # service.http is now a httplib2shim.Http object.

Unsupported Features
--------------------

* Arguments to the `Http` constructor will be accepted, but may not make a difference. For instance, ``ca_certs`` will have no effect. Instead, pass a ``urllib3.Pool`` instance ``http = httplib2shim.Http(pool=my_pool)``.
* `Http.add_certificate` is a no-op and will warn.
* Probably others, pull requests are welcome to complete the functionality.


Contributing changes
--------------------

-  See `CONTRIBUTING.md`_

Licensing
---------

- MIT - See `LICENSE`_

.. _LICENSE: https://github.com/GoogleCloudPlatform/httplib2shim/blob/master/LICENSE
.. _CONTRIBUTING.md: https://github.com/GoogleCloudPlatform/httplib2shim/blob/master/CONTRIBUTING.md

.. |Build Status| image:: https://travis-ci.org/GoogleCloudPlatform/httplib2shim.svg
   :target: https://travis-ci.org/GoogleCloudPlatform/httplib2shim
.. |Coverage Status| image:: https://coveralls.io/repos/GoogleCloudPlatform/httplib2shim/badge.svg?branch=master&service=github
   :target: https://coveralls.io/github/GoogleCloudPlatform/httplib2shim?branch=master
.. |PyPI Version| image:: https://img.shields.io/pypi/v/httplib2shim.svg
   :target: https://pypi.python.org/pypi/httplib2shim
