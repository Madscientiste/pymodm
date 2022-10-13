.. |ss| raw:: html

    <strike>

.. |se| raw:: html

    </strike>


======
PyMODM
======

**Please don't use this fork on your projects.**

**MongoDB has paused the development of PyMODM.** If there are any users who want
to take over and maintain this project, or if you just have questions, please respond
to `this forum post <https://developer.mongodb.com/community/forums/t/updates-on-pymodm/9363>`_.

================

A generic ODM around PyMongo_, the MongoDB Python driver. PyMODM works on Python
2.7 as well as Python 3.3 and up. To learn more, you can browse the `official
documentation`_ or take a look at some `examples`_.

.. _PyMongo: https://pypi.python.org/pypi/pymongo
.. _official documentation: http://pymodm.readthedocs.io/en/stable
.. _examples: https://github.com/mongodb/pymodm/tree/master/example

Why PyMODM?
===========

PyMODM is a "core" ODM, meaning that it provides simple, extensible
functionality that can be leveraged by other libraries to target platforms like
Django. At the same time, PyMODM is powerful enough to be used for developing
applications on its own.

|ss| Because MongoDB engineers are involved in developing
and maintaining the project, PyMODM will also be quick to adopt new MongoDB
features. |se|

Support / Feedback
==================

For issues with, questions about, or feedback for PyMODM, please look into
our `support channels <http://www.mongodb.org/about/support>`_. Please do not
email any of the PyMODM developers directly with issues or questions -
you're more likely to get an answer on the `MongoDB Community Forums <https://developer.mongodb.com/community/forums/tags/c/drivers-odms-connectors/7/pymodm-odm>`_.


Example
=======

Here's a basic example of how to define some models and connect them to MongoDB:

.. code-block:: python

  from pymongo import TEXT
  from pymongo.operations import IndexModel
  from pymodm import connect, fields, MongoModel, EmbeddedMongoModel


  # Connect to MongoDB first. PyMODM supports all URI options supported by
  # PyMongo. Make sure also to specify a database in the connection string:
  connect('mongodb://localhost:27017/myApp')


  # Now let's define some Models.
  class User(MongoModel):
      # Use 'email' as the '_id' field in MongoDB.
      email = fields.EmailField(primary_key=True)
      fname = fields.CharField()
      lname = fields.CharField()


  class BlogPost(MongoModel):
      # This field references the User model above.
      # It's stored as a bson.objectid.ObjectId in MongoDB.
      author = fields.ReferenceField(User)
      title = fields.CharField(max_length=100)
      content = fields.CharField()
      tags = fields.ListField(fields.CharField(max_length=20))
      # These Comment objects will be stored inside each Post document in the
      # database.
      comments = fields.EmbeddedModelListField('Comment')

      class Meta:
          # Text index on content can be used for text search.
          indexes = [IndexModel([('content', TEXT)])]

  # This is an "embedded" model and will be stored as a sub-document.
  class Comment(EmbeddedMongoModel):
      author = fields.ReferenceField(User)
      body = fields.CharField()
      vote_score = fields.IntegerField(min_value=0)


  # Start the blog.
  # We need to save these objects before referencing them later.
  han_solo = User('mongoblogger@reallycoolmongostuff.com', 'Han', 'Solo').save()
  chewbacca = User(
      'someoneelse@reallycoolmongostuff.com', 'Chewbacca', 'Thomas').save()


  post = BlogPost(
      # Since this is a ReferenceField, we had to save han_solo first.
      author=han_solo,
      title="Five Crazy Health Foods Jabba Eats.",
      content="...",
      tags=['alien health', 'slideshow', 'jabba', 'huts'],
      comments=[
          Comment(author=chewbacca, body='Rrrrrrrrrrrrrrrr!', vote_score=42)
      ]
  ).save()


  # Find objects using familiar MongoDB-style syntax.
  slideshows = BlogPost.objects.raw({'tags': 'slideshow'})

  # Only retrieve the 'title' field.
  slideshow_titles = slideshows.only('title')

  # u'Five Crazy Health Foods Jabba Eats.'
  print(slideshow_titles.first().title)
