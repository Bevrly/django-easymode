:mod:`easymode.middleware`
==========================

Google Analytics
----------------

Easymode has middleware to support caching in combination with google analytics.
Google analytics updates a session cookie on each request. Because django's
:class:`~django.contrib.sessions.middleware.django.contrib.sessions.middleware.SessionMiddleware` places cookie in it's vary header, *you will save every single
request to the cache* if you use it.

.. autoclass:: easymode.middleware.NoVaryOnCookieSessionMiddleWare

Internationalization related middleware
---------------------------------------

.. automodule:: easymode.middleware
    :members: LocaleFromUrlMiddleWare, LocaliseUrlsMiddleware, ShortLocaleFromUrlMiddleWare, ShortLocaleLocaliseUrlsMiddleware