Django OpenID Connect (OIDC) authentication provider
====================================================

This module makes it easy to integrate OpenID Connect as an authentication source in a Django project.

Behind the scenes, it uses Roland Hedberg's great pyoidc library.

Modified by JHUAPL BOSS to support Python3

Modified by Thomas Frössman with fixes and additional modifications.

Quickstart
----------

Install djangooidc::

    # Latest code - unstable!
    pip install git+https://github.com/{desiredforkname}/django-oidc.git

(replace {desiredforkname} by the github username; find the fork which suit your needs, or just copy the name from your browser location field).

Then to use it in a Django project, add this to your urls.py::

    url(r'^openid/', include('djangooidc.urls')),


Then add the following items to your settings.py:

* add `'djangooidc.backends.OpenIdConnectBackend'` to AUTHENTICATION_BACKENDS **after** the default
  `'django.contrib.auth.backends.ModelBackend'`
* set LOGIN_URL = 'openid'
* add the specific OIDC parameters (change the absolute URLs to yours)::

    # Information used when registering the client, this may be the same for all OPs
    # Ignored if auto registration is not used.
    OIDC_DYNAMIC_CLIENT_REGISTRATION_DATA = {
        "application_type": "web",
        "contacts": ["ops@example.com"],
        "redirect_uris": ["http://localhost:8000/openid/callback/login/", ],
        "post_logout_redirect_uris": ["http://localhost:8000/openid/callback/logout/", ]
    }

    # Default is using the 'code' workflow, which requires direct connectivity from your website to the OP.
    OIDC_DEFAULT_BEHAVIOUR = {
        "response_type": "code",
        "scope": ["openid", "profile", "email", "address", "phone"],
    }

The configuration above is enough to use OIDC providers (OP) that support discovery and self client registration.
In addition, you may want to use a specific OpenID Connect provider that is not auto-discoverable. This is done
by adding items to the OIDC_PROVIDERS dictionary. See full documentation for parameter names.

For example, an Azure AD OP would be::

    OIDC_PROVIDERS = {
        "Azure Active Directory": {
            "srv_discovery_url": "https://sts.windows.net/aaaaaaaa-aaaa-1111-aaaa-xxxxxxxxxxxxx/",
            "behaviour": OIDC_DEFAULT_BEHAVIOUR,
            "client_registration": {
                "client_id": "your_client_id",
                "client_secret": "your_client_secret",
                "redirect_uris": ["http://localhost:8000/openid/callback/login/"],
                "post_logout_redirect_uris": ["http://localhost:8000/openid/callback/logout/"],
            }
        }
    }


You may now test the authentication by going to (on the development server) http://localhost:8000/openid/login or to any
of your views that requires authentication.

Using a private key jwt for client authentication
-------------------------------------------------
If you are using private keys for client authentication with the OP, you can specify it like::

    OIDC_PROVIDERS = {
        "mitreid": {
            "srv_discovery_url": "https://mitreid.org/",
            "behaviour": OIDC_DEFAULT_BEHAVIOUR,
            "client_registration": {
                "client_id": "your_client_id",
                "redirect_uris": ["http://localhost:8000/openid/callback/login/"],
                'token_endpoint_auth_method': ['private_key_jwt'],
                "enc_kid": "rsa_test",
                "keyset_jwk_file": "file://keys/keyset.jwk"
            }
        }
    }

In this case keys/keyset.jwk is the full keyset (public and private keys) used when registering the client with the OP
manually. (I.E. you've provided the OP with the public key.)

Features
--------

* Ready to use Django authentication backend
* No models stored in database - just some configuration in settings.py to keep it simple
* Fully integrated with Django's internal accounts and permission system
* Support for all OIDC workflows: Authorization Code flow, Implicit flow, Hybrid flow. Don't worry if you don't know
  what these are - the package comes with great defaults.
* Includes logout at the provider level
