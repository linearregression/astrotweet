==================================================================
astrotweet - Work with AstroBetter's Index of Tweeting Astronomers
==================================================================

This is a project to build on `AstroBetter's index of tweeting astronomers. <http://www.astrobetter.com/wiki/tiki-index.php?page=Astronomers+on+Twitter/>`_ 
Some things that we *will* eventually do include:

- building and updating a twitter list of astronomers
- machine-readable analytics of astronomy tweeters, including number of tweets or followers.


Installation
------------

Download the code and run ``setup.py install``.

You'll need the following third-party packages: ``Cliff``, ``requests``, ``twitter``, ``pymongo`` and ``networkx``.
Setuptools should install these automatically.
To check your installation run::

    astrotweet --help


Getting Twitter Credientials
----------------------------

``astrotweet`` uses Twitter's v1.1 API.
This means that you need to get a set of application credientials to run the code.
It's easy:

1. `Register the application. <https://dev.twitter.com/apps/new/>`_ Make up an application name, description, etc.. Don't share the consumer key or secret with anyone.
2. Go ahead and press the button to create your OAuth access token. Again, don't share this publicly.
3. Store the credentials into a file at ``$HOME/.astrotweet.json``; the format is::

    {
        "consumer_key": "your_consumer_key,
        "consumer_secret": "your_consumer_secret",
        "oauth_token": "your_access_token",
        "oauth_token_secret": "your_access_token_secret"
    }


Commands
--------

``astrotweet`` is a command line app built around the philosophy of subcommands, much like ``git``.
Here are the commands currently included in ``astrotweet``:

- ``astrotweet summary`` - will output a summary table (tab-separated) of astrotweeters to the file ``astrotweeters.csv``. Columns include 'real' name, follower counts, etc..
- ``astrotweet build`` - will build a MongoDB collection of user data (see schema below).

  - Add in the ``--followers`` and ``--friends`` flags to also grab friend and follower lists. Note that this will take a while because of Twitter's rate limits.
  - The ``--cliques`` flag creates a MongoDB collection of mutual-follower cliques using ``networkx``; useful if follower and friend data has been grabbed.

You can get a overview of ``astrotweet``'s commands by running ``astrotweet --help``, and using the ``astrotweet help cmd_name`` for help specific to each command (e.g., ``astrotweet help build``).


Schema of user documents in MongoDB
-----------------------------------

The ``astrotweet build`` command loads user data into an ``astrotweet.users`` collection in a MongoDB database (see below for how to start up your own MongoDB server).
The schema for these documents is:

- ``_id``: twitter user ID, as a string. Also the formal document ID.
- ``screen_name``: user's twitter handle, without the '@'. I.e., I'm ``jonathansick``.
- ``id``: twitter user ID, as an integer
- ``description``: description, written by the user
- ``profile_image_url``: URL of user's twitter avatar (jpg image)
- ``url``: homepage for this user (NULL if none specified)
- ``followers_count``: number of people following this user
- ``friends_count``: number of people that this user follows
- ``statuses_count``: number of tweets by user
- ``location``: string location of user (not consistent, nor always relevant)
- ``lang``: user's language code
- ``created_at``: datetime when user created account
- ``updated_at``: datetime when user's data was added to MongoDB
- ``listed_count``: number of twitter lists this user appears in
- ``friend_ids``: list of ``str_id`` of people this user is follower. Built with ``astrotweet build --friends`` flag.
- ``follower_ids``: list of ``str_id`` of people following this user. Built with ``astrotweet build --followers`` flag.


Schema of the clique collection in MongoDB
------------------------------------------

Every clique built with ``astrotweet build --cliques`` is stored in a ``astrotweet.cliques`` collection in a MongoDB database, alongside the user collection.
The schema is bare bones:

- ``_id``: auto-generated ObjectId
- ``members``: list of ``str_ids`` of users in the clique
- ``size``: number of users in the clique (i.e., length of ``members``)

Note that this collection, and its documents, are erased and rebuilt every time ``astrotweet build --cliques`` is run: **do not store custom data in these documents!**


Running MongoDB
---------------

For the commands that require MongoDB persistence, it is easy to setup a local MongoDB server.
First, `download the lastest MongoDB binaries <http://www.mongodb.org/downloads/>`_ and install them into your path.

Then create a directory to store the database and boot up the ``mongod`` server with::

    mkdir -p $HOME/mongodata
    mongod --dbpath $HOME/mongodata --fork --logpath $HOME/mongo.log

You can later kill the ``mongod`` process to safely shutdown the DB server.


About
-----

This an experiment by Jonathan Sick. Tweet: @jonathansick
