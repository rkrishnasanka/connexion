Security
========

OAuth 2 Authentication and Authorization
----------------------------------------

Connexion supports one of the three OAuth 2 handling methods.
With Connexion, the API security definition **must** include a
``x-tokenInfoFunc`` or set ``TOKENINFO_FUNC`` env var.

``x-tokenInfoFunc`` must contain a reference to a function
used to obtain the token info. This reference should be a string using
the same syntax that is used to connect an ``operationId`` to a Python
function when routing. For example, an ``x-tokenInfoFunc`` with a value of
``auth.verifyToken`` would pass the user's token string to the function
``verifyToken`` in the module ``auth.py``. The referenced function accepts
a token string as argument and should return a dict containing a ``scope``
field that is either a space-separated list or an array of scopes belonging to
the supplied token. This list of scopes will be validated against the scopes
required by the API security definition to determine if the user is authorized.
You can supply a custom scope validation func with ``x-scopeValidateFunc``
or set ``SCOPEVALIDATE_FUNC`` env var, otherwise default scope validation function
``connexion.security.security_handler_factory.validate_scope`` will be used automatically.


The recommended approach is to return a dict which complies with
`RFC 7662 <rfc7662_>`_. Note that you have to validate the ``active``
or ``exp`` fields etc. yourself.

The Token Info response will be passed in the ``token_info`` argument to the handler
function. The ``sub`` property of the Token Info response will be passed in the ``user``
argument to the handler function.

Deprecated features, retained for backward compatibility:

- As alternative to ``x-tokenInfoFunc``, you can set ``x-tokenInfoUrl`` or
  ``TOKENINFO_URL`` env var. It must contain a URL to validate and get the token
  information which complies with `RFC 6749 <rfc6749_>`_.
  When both ``x-tokenInfoUrl`` and ``x-tokenInfoFunc`` are used, Connexion
  will prioritize the function method. Connexion expects the authorization
  server to receive the OAuth token in the ``Authorization`` header field in the
  format described in `RFC 6750 <rfc6750_>`_ section 2.1. This aspect represents
  a significant difference from the usual OAuth flow.
- ``scope`` field can also be named ``scopes``.
- ``sub`` field can also be named ``uid``.

You can find a `minimal OAuth example application`_ showing the use of
``x-tokenInfoUrl``, and `another OAuth example`_ showing the use of
``x-tokenInfoFunc`` in Connexion's "examples" folder.

.. _minimal OAuth example application: https://github.com/zalando/connexion/tree/main/examples/swagger2/oauth2
.. _another OAuth example: https://github.com/zalando/connexion/tree/main/examples/swagger2/oauth2_local_tokeninfo

Basic Authentication
--------------------

With Connexion, the API security definition **must** include a
``x-basicInfoFunc`` or set ``BASICINFO_FUNC`` env var. It uses the same
semantics as for ``x-tokenInfoFunc``, but the function accepts three
parameters: username, password and required_scopes.

You can find a `minimal Basic Auth example application`_ in Connexion's "examples" folder.

.. _oauth scope: https://oauth.net/2/scope/
.. _minimal Basic Auth example application: https://github.com/zalando/connexion/tree/main/examples/openapi3/basicauth

ApiKey Authentication
---------------------

With Connexion, the API security definition **must** include a
``x-apikeyInfoFunc`` or set ``APIKEYINFO_FUNC`` env var. It uses the same
semantics as for ``x-basicInfoFunc``, but the function accepts two
parameters: apikey and required_scopes.

You can find a `minimal API Key example application`_ in Connexion's "examples" folder.

Bearer Authentication (JWT)
---------------------------

With Connexion, the API security definition **must** include a
``x-bearerInfoFunc`` or set ``BEARERINFO_FUNC`` env var. It uses the same
semantics as for ``x-tokenInfoFunc``, but the function accepts one parameter: token.

You can find a `minimal JWT example application`_ in Connexion's "examples/openapi3" folder.

Multiple Authentication Schemes
-------------------------------

With Connexion, it is also possible to combine multiple authentication schemes
as described in the `OpenAPI specification`_. When multiple authentication
schemes are combined using logical AND, the ``token_info`` argument will
consist of a dictionary mapping the names of the security scheme to their
corresponding ``token_info``.

Multiple OAuth2 security schemes in AND fashion are not supported.

.. _OpenAPI specification: https://swagger.io/docs/specification/authentication/#multiple

Deploying Authentication
------------------------

Some production hosting environments, such as Apache with modwsgi, do not by default pass
authentication headers to WSGI applications.  Therefore, to allow connexion to handle
authentication, you will need to enable passthrough.

Instructions for `enabling authentication passthrough in modwsgi`_ are available as
part of the `modwsgi documentation`_.

HTTPS Support
-------------

When specifying HTTPS as the scheme in the API YAML file, all the URIs
in the served Swagger UI are HTTPS endpoints. The problem: The default
server that runs is a "normal" HTTP server. This means that the
Swagger UI cannot be used to play with the API. What is the correct
way to start a HTTPS server when using Connexion?

One way, `described by Flask`_, looks like this:

.. code-block:: python

   from OpenSSL import SSL
   context = SSL.Context(SSL.SSLv23_METHOD)
   context.use_privatekey_file('yourserver.key')
   context.use_certificate_file('yourserver.crt')

   app.run(host='127.0.0.1', port='12344',
           debug=False/True, ssl_context=context)

However, Connexion doesn't provide an ssl_context parameter. This is
because Flask doesn't, either--but it uses ``**kwargs`` to send the
parameters to the underlying `werkzeug`_ server.

.. _rfc6750: https://tools.ietf.org/html/rfc6750
.. _rfc6749: https://tools.ietf.org/html/rfc6749
.. _rfc7662: https://tools.ietf.org/html/rfc7662
.. _minimal API Key example application: https://github.com/zalando/connexion/blob/main/examples/openapi3/apikey
.. _minimal JWT example application: https://github.com/zalando/connexion/tree/main/examples/openapi3/jwt
.. _enabling authentication passthrough in modwsgi: https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIPassAuthorization.html
.. _modwsgi documentation: https://modwsgi.readthedocs.io/en/develop/index.html
