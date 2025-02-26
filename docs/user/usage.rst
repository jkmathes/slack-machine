.. _usage:

Using Slack Machine
===================

Once you have installed Slack Machine, configuring and starting your bot is easy:

1. Create a directory for your Slack Machine bot: ``mkdir my-slack-bot && cd my-slack-bot``
2. Add a ``local_settings.py`` file to your bot directory: ``touch local_settings.py``
3. Create a Bot User for your Slack team: https://my.slack.com/services/new/bot (take note of your API token)
4. Add the Slack API token to your ``local_settings.py`` like this:

.. code-block:: python

    SLACK_API_TOKEN = 'xox-my-slack-token'

5. Start the bot with ``slack-machine``
6. \...
7. Profit!

Configuring Slack Machine
-------------------------

All the configuration for your bot lives in the ``local_settings.py`` in the root of your bot
directory. The core of Slack Machine, and the built-in plugins, only need a ``SLACK_API_TOKEN``
to function.

You can override the log level by setting ``LOGLEVEL``. By default this is set to ``"ERROR"``.

If you want to use Slack Machine behind a proxy, you can use ``HTTP_PROXY`` and ``HTTPS_PROXY``.
Be cautious, only http proxy is supported for now.

If you find you have issues with Slack Machine disconnecting, try enabling the keep alive
feature by setting ``KEEP_ALIVE`` to an integer (interval in seconds to send keep alive pings).

Using environment variables for configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For some configuration, it can be a security consideration not to store them in your source code
(i.e. ``local_settings.py``) Slack Machine allows you to provide any setting - both built-in and
for plugins - as environment variables. This is done by prefixing the setting name with ``SM_``.
Example: ``SM_SLACK_API_TOKEN`` as environment variable can be used to set the ``SLACK_API_TOKEN``
setting instead of having to put it in the ``local_settings.py``.

Setting aliases
~~~~~~~~~~~~~~~

The ``ALIASES`` configuration setting allows the bot to respond to a trigger symbol instead of a
direct @botname.

Example:

If ``ALIASES='!,%'`` was set in ``local_settings.py`` then the bot would respond to the following
phrases::

    @botname release the hounds
    !release the hounds
    %release the hounds

Enabling plugins
""""""""""""""""

Slack Machine comes with a few simple built-in plugins:

- **HelloPlugin**: responds in kind when users greet the bot with "hello" or "hi" (only when the
  bot is mentioned)
- **PingPongPlugin**: responds to "ping" with "pong" and vice versa (listens regardless of mention)
- **EchoPlugin**: replies to any message the bot hears, with exactly the same message. The bot will
  reply to the same channel the original message was heard in

By default, **HelloPlugin** and **PingPongPlugin** are enabled.

You can specify which plugins Slack Machine should load, by setting the ``PLUGINS`` variable in
``local_settings.py`` to a list of fully qualified classes or modules that contain plugins.
You can either point to a plugin class directly, or to a module containing one or more plugins.

For example, to enable all built-in Slack Machine plugins, your ``local_settings.py`` would look like this:

.. code-block:: python

    SLACK_API_TOKEN = 'xoxb-my-slack-token'
    PLUGINS = [
        'machine.plugins.builtin.general.PingPongPlugin',
        'machine.plugins.builtin.general.HelloPlugin',
        'machine.plugins.builtin.debug.EventLoggerPlugin',
        'machine.plugins.builtin.debug.EchoPlugin'
    ]

Or is you want import them by the modules they're in:

.. code-block:: python

    SLACK_API_TOKEN = 'xox-my-slack-token'
    PLUGINS = [
        'machine.plugins.builtin.general',
        'machine.plugins.builtin.debug'
    ]

Slack Machine can load any plugin that is on the Python path. This means you can load any plugin that
is installed in the same virtual environment you installed Slack Machine in. And as a convenience,
Slack Machine will also add the directory you start Slack Machine from, to your Python path.

.. _storage options:

Choosing storage
""""""""""""""""

Slack Machine provides persistent storage, which can be used by plugins to store data of any kind.
Slack Machine supports different *backends* for storage, so you can choose one that best fits your
needs and existing infrastructure. You can configure which backend to use, by setting the
``STORAGE_BACKEND`` variable in ``local_settings.py`` to the fully qualified class of the chosen
storage backend.

Out of the box, Slack Machine provides 2 options for storage backend:

- **in-memory** (*default*): this backend will store all data in-memory, which is great for testing because
  it doesn't have any external dependencies. **Does not persist data between restarts**

  *Class*: ``machine.storage.backends.memory.MemoryStorage``

- **Redis**: this backend stores data in `Redis`_. Redis is a very fast key-value store that is super
  easy to install and operate. This backend is recommended, because it will persist data between restarts.
  The Redis backend requires you to provide a URL to your Redis instance by setting the ``REDIS_URL``
  variable in ``local_settings.py``. The URL should have the following format:

  ``redis://<host>:<port>[/<db>]``

  Where ``db`` is optional and sets the database number (*0* by default)

  Optional parameters:

  - ``REDIS_MAX_CONNECTIONS``: maximum number of connections Slack Machine can make to your Redis instance
  - ``REDIS_KEY_PREFIX``: the prefix Slack Machine uses for keys (``SM`` by default, so "key1" gets
    stored under ``SM:key1``)

  *Class*: ``machine.storage.backends.redis.RedisStorage``

- **HBase**: this backend stores data in `HBase`_. HBase is a columnar store. This backend is for
  advanced users only. You should only use it if you already have a HBase cluster running and cannot
  use Redis for some reason. This backend requires 2 variables to be set in your ``local_settings.py``:
  ``HBASE_URL`` and ``HBASE_TABLE``.

  *Class*: ``machine.storage.backends.hbase.HBaseStorage``
    
- **DynamoDB**: this backend stores data in `DynamoDB`. `DynamoDB`_ is a managed NoSQL datastore on 
  AWS that, among other things, allows for easy persistance of objects by key. The DynamoDB backend
  requires either a set of valid AWS account credentials, or a locally running DynamoDB test bed, 
  such as the one included in `localstack`_. This backend
  requires only the environment variables or path-based AWS credentials that are normally used to
  access AWS endpoints. The following are optional parameters that can be set in your ``local_settings.py``
  or `SM_` environment variable slack-machine settings:
  
  Optional parameters:
  
  - ``DYNAMODB_ENDPOINT_URL``: specifies an optional alternate endpoint, for local bot testing
  - ``DYNAMODB_KEY_PREFIX``: an optional prefix to use within the key lookup. Defaults to `SM:`
  - ``DYNAMODB_TABLE_NAME``: specifies the table to use in DynamoDB. Defaults to `slack-machine-state`
  - ``DYNAMODB_CREATE_TABLE``: optionally -create- the table to be used in DynamoDB. Defaults to `False`
  - ``DYNAMODB_CLIENT``: if custom configuration is needed for the DynamoDB client, an optional `boto3`_ client can be specified here
  
  *Class*: ``machine.storage.backends.dynamodb.DynamoDBStorage``

So if, for example, you want to configure Slack Machine to use Redis as a storage backend, with your Redis
instance running on *localhost* on the default port, you would add this to your ``local_settings.py``:

.. code-block:: python

    STORAGE_BACKEND = 'machine.storage.backends.redis.RedisStorage'
    REDIS_URL = redis://localhost:6379'

.. _Redis: https://redis.io/

.. _HBase: https://hbase.apache.org/

.. _DynamoDB: https://aws.amazon.com/dynamodb/

.. _localstack: https://github.com/localstack/localstack

.. _boto3: https://aws.amazon.com/sdk-for-python

That's all there is to it!
