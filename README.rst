=======
Aldjemy
=======

Base
----

This is a small package to integrate SQLAlchemy into an existing Django
project.  The primary use case of this package is building complex queries that
are not possible with the Django ORM.

You need to include aldjemy at the end of ``INSTALLED_APPS``. When models are
imported, aldjemy will read all models and add an ``sa`` attribute to them.
The ``sa`` attribute is a class, mapped to a SQLAlchemy ``Table``.

Internally, aldjemy generates tables from Django models. This is an important
distinction from the standard decision of using SQLAlchemy reflection.

Code example::

    User.sa.query().filter(User.sa.username=='Brubeck')

M2M sample::

    User.sa.query().join(User.sa.groups).filter(Group.sa.name=="GROUP_NAME")

Explicit joins are part of SQLAlchemy philosophy, so aldjemy can't get you
exactly the same experience as Django.  But aldjemy is not positioned as a
Django ORM drop-in replacement. It's a helper for special situations.

We have some stuff in the aldjemy cache too::

    from aldjemy import core
    core.Cache.models # All generated models
    core.get_tables() # All tables, and M2M tables too

You can use this stuff if you need - maybe you want to build queries with
tables, or something like this.


Settings
--------

You can add your own field types to map django types to sqlalchemy ones with
the ``ALDJEMY_DATA_TYPES`` setting.  It must be a ``dict``. Keys are the result
of ``field.get_internal_type()``, values must be a one arg function.  For
examples, look at the source of the  ``aldjemy.types`` module.
  
Also it is possible to extend/override the list of supported SQLALCHEMY engines
using the ``ALDJEMY_ENGINES`` setting.  It should be a ``dict``.  The keys are
the substring after the last dot from the Django database engine setting (e.g.
``sqlite3`` from ``django.db.backends.sqlite3``), values are the SQLAlchemy
driver which will be used for that connection (e.g. ``sqlite``,
``sqlite+pysqlite``).  It could be helpful if you want to use
``django-postgrespool``.


Mixins
------

Often, Django models have helper functions and properties that help to
represent the model's data (``__unicode__``), or represent some model based
logic.

To integrate them with aldjemy models you can put these methods into a separate
mixin::

    class TaskMixin(object):
        def __unicode__(self):
            return self.code

    class Task(TaskMixin, models.Model):
        aldjemy_mixin = TaskMixin
        code = models.CharField(_('code'), max_length=32, unique=True)

Voilà! You can use ``unicode`` on aldjemy classes, because this mixin will be
mixed into the generated aldjemy model.

If you want to expose all methods and properties without creating a separate
mixin class, you can use the ``aldjemy.meta.AldjemyMeta`` metaclass::

    from aldjemy.meta import AldjemyMeta

    class Task(models.Model):
        code = models.CharField(_('code'), max_length=32, unique=True)

        def __unicode__(self):
            return self.code

        __metaclass__ = AldjemyMeta

The result is same as with the example above, only you didn't need to
create the mixin class at all.

Also note that with Python 3, the syntax is a bit different::

    class Task(models.Model, metaclass=AldjemyMeta):
        code = models.CharField(_('code'), max_length=32, unique=True)

        def __str__(self):
            return self.code
