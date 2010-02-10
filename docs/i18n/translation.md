.. _translation_of_contents:

Translation of database content
===============================

When using the :ref:`i18n <internationalization_of_models>` and 
:ref:`l10n <localization_of_admin>` features of easymode, you can use gettext's
standard translation features to translate all the database content.

If the :ref:`master_site` directive is set to True, every time a model decorated
with ``I18n`` is saved, easymode will add an entry to the corresponding gettext 
catalog. (for all the options related to the location of the catalogs please 
refer to :doc:`/settings`)

For each language in your ``LANGUAGES`` directive, a catalog will be created.
This way you can translate all the content using something like 
`poedit <http://www.poedit.net/>`_ or 
`rosetta <http://code.google.com/p/django-rosetta/>`_. This is especially
convenient when a new site is created, for the first *big batch* of translations.

For modifications afterward, you can just use the admin interface, which will
show the translations from the gettext catalog if they exist.

Translation mechanism explained
-------------------------------

It is important to realise, that allthough you can make translations using gettext,
the catalog is not the only place translations are stored. The 
:ref:`I18n decorator<internationalization_of_models>` not only registers a model
for catalog management, it also modifies the model.

suppose we have a model as follows:

::

    I18n(bar)
    class Foo(models.Model):
        bar = models.CharField(max_length=255)
        foobar = models.TextField()

Normally the database would look like this::

    CREATE TABLE "foobar_foo" (
        "id" integer NOT NULL PRIMARY KEY,
        "bar" varchar(255) NOT NULL,
        "foobar" text NOT NULL
    )

The :ref:`I18n decorator<internationalization_of_models>` modiefies the model,
given we've got both 'en' and 'yx' in out ``LANGUAGES`` directive this is what
the model would look like on the database end::

    CREATE TABLE "foobar_foo" (
        "id" integer NOT NULL PRIMARY KEY,
        "bar_en" varchar(255) NULL,
        "bar_yx" varchar(255) NULL,
        "foobar" text NOT NULL
    )

On the model end you would not see this, because you will still access ``bar`` 
like this::

    >>> m = Foo.objects.get(pk=1)
    >>> m.bar = 'hello'
    >>> print m.bar
    hello

Any field that is internationalized using the 
:ref:`I18n decorator<internationalization_of_models>` will always return the 
field in the current languge, both on read and on write.

Database is bigger than gettext
-------------------------------

**Only when a field is empty** (``None``) **in the database for the current language, the
gettext catalog will be consulted for a translation**

This way, a model has exactly the same semantics as before, in that we can read
and write to the property, the way we defined it in it's declaration. We 
still get the gettext goodies, which is nice when large ammounts of text must be
translated. 

If the gettext catalog would be the only place where the translations
would be stored, having proper write semantics would become very difficult.

Example::

    >>> from django.utils.translation import activate
    
    >>> m = Foo()
    >>> m.bar = 'hello'
    >>> m.bar
    'hello'
    >>> activate('yx')
    >>> m.bar
    'hello'
    >>> m.bar = 'xy says hello'
    >>> m.bar
    'xy says hello'
    >>> activate('en')
    >>> m.bar
    'hello'

What you'll notice is that ``m.bar`` is allready available in the language 'yx'
even though we did't specify it's value yet. This is because the normal behaviour
of gettext is to return the ``msgid`` if the ``msgstr`` is not yet available. 
This is because the value for ``m.bar`` in langugae 'yx' was resolved as follows:

* see if the database value bar_yx is not null, if so return bar_yx
* see if the ``msgstr`` for 'hello' (The value of ``m.bar`` in the 
  :ref:`msgid_langugae`) exists if so return ugettext('hello')
* otherwise return the value in the :ref:`fallback language <fallback_langugaes>`
    
Importing translations is implicit
----------------------------------

One thing that follows from the mechanics as described above, is that there is
no need to explicitly import translations from gettext catalogs into the database.

Importing does take place however, each time a model is saved in the admin, the
translations are written to the database. 

This is because the translations from the gettext catalog *ARE* displayed in the 
admin, which means they *ARE* present in the form, but since the database column 
itself is *EMPTY* it will be marked as a change and written to the appropriate 
field.

This implicit import could pose a problem. If for example a model was edited in the
admin, *BEFORE* the gettext catalog was properly translated and imported, it could
be that the wrong value, from some :ref:`fallback language <fallback_langugaes>`
got written to the database. Because the database is bigger than gettext catalog
the new translation would never show up.

This inconvenience can be resolved using the :ref:`easy_reset_language` command
    