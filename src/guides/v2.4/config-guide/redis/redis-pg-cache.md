---
group: configuration-guide
title: Use Redis for default cache
functional_areas:
  - Configuration
  - System
  - Setup
---

Magento provides command line options to configure the Redis page and default caching. Although you can configure caching by editing the `<Magento install dir>app/etc/env.php` file, using the command line is the recommended method, especially for initial configurations. The command line provides validation, ensuring the configuration is syntactically correct.

You must [install Redis]({{ page.baseurl }}/config-guide/redis/config-redis.html#config-redis-install) before continuing.

## Configure Redis default caching {#config-redis-config}

Run the `setup:config:set` command and specify parameters that specific to Redis default caching.

```bash
bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-<parameter_name>=<parameter_value>...
```

where

`--cache-backend=redis` enables the Redis default caching. If this feature has already been enabled, omit this parameter.

`--cache-backend-redis-<parameter_name>=<parameter_value>` is a list of key/value pairs that configure the default caching:

|Command line parameter|Parameter|Meaning|Default value|
|--- |--- |--- |--- |
|cache-backend-redis-server|server|Fully qualified hostname, IP address, or an absolute path to a UNIX socket. The default value of 127.0.0.1 indicates Redis is installed on the Magento server.|127.0.0.1|
|cache-backend-redis-port|port|Redis server listen port|6379|
|cache-backend-redis-db|database|Required if you use Redis for both the default and full-page cache. You must specify the database number of one of the caches; the other cache uses 0 by default.<br/><br/>Important: If you use Redis for more than one type of caching, the database numbers must be different. It is recommended that you assign the default caching database number to 0, the page caching database number to 1, and the session storage database number to 2.|0|
|cache-backend-redis-password|password|Configuring a Redis password enables one of its built-in security features: the auth command, which requires clients to authenticate to access the database. The password is configured directly in Redis's configuration file, `/etc/redis/redis.conf`.||

### Example command

The following example enables Redis default caching, sets the host to `127.0.0.1` and assigns the database number to 0. Redis uses default values for all other parameters.

```bash
bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=127.0.0.1 --cache-backend-redis-db=0
```

## Configure Redis page caching

To configure Redis page caching on Magento, run the `setup:config:set` command with additional parameters.

```bash
bin/magento setup:config:set --page-cache=redis --page-cache-redis-<parameter_name>=<parameter_value>...
```

where

`--page-cache=redis` enables Redis page caching. If this feature has already been enabled, omit this parameter.

`--page-cache-redis-<parameter_name>=<parameter_value>` is a list of parameter/value pairs that configure page caching:

|Command line parameter|Parameter|Meaning|Default value|
|--- |--- |--- |--- |
|page-cache-redis-server|server|Fully qualified hostname, IP address, or an absolute path to a UNIX socket. The default value of 127.0.0.1 indicates Redis is installed on the Magento server.|127.0.0.1|
|page-cache-redis-port|port|Redis server listen port|6379|
|page-cache-redis-db|database|Required if you use Redis for both the default and full page cache. You must specify the database number of one of the caches; the other cache uses 0 by default.<br/>Important: If you use Redis for more than one type of caching, the database numbers must be different. It is recommended that you assign the default caching database number to 0, the page caching database number to 1, and the session storage database number to 2.|0|
|page-cache-redis-password|password|Configuring a Redis password enables one of its built-in security features: the auth command, which requires clients to authenticate to access the database. Configure the password within the Redis configuration file, /etc/redis/redis.conf.||

### Example command

The following example enables Redis page caching, sets the host to `127.0.0.1` and assigns the database number to 1. All other parameters are set to the default value.

```bash
bin/magento setup:config:set --page-cache=redis --page-cache-redis-server=127.0.0.1 --page-cache-redis-db=1
```

## Results

As a result of the two example commands, Magento adds lines similar to the following to `<Magento install dir>app/etc/env.php`:

```php
'cache' => [
    'frontend' => [
        'default' => [
            'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
            'backend_options' => [
                'server' => '127.0.0.1',
                'database' => '0',
                'port' => '6379'
            ],
        ],
        'page_cache' => [
            'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
            'backend_options' => [
                'server' => '127.0.0.1',
                'port' => '6379',
                'database' => '1',
                'compress_data' => '0'
            ]
        ]
    ]
],
```

## Using AWS ElastiCache with your EC2 instance

As of Magento 2.4.3, Magento instances hosted on Amazon EC2 may use an AWS ElastiCache in place of a local Redis instance.

{:.bs-callout-warning}
This section only works for Magento instances running on Amazon EC2 VPCs. It does not work for on-premises installations.

### Configure a Redis cluster

After [setting up a Redis cluster on AWS](https://aws.amazon.com/getting-started/hands-on/setting-up-a-redis-cluster-with-amazon-elasticache/), configure the EC2 instance to use the ElastiCache.

1. [Create an ElastiCache Cluster](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/set-up.html) in the same region and VPC of the EC2 instance.
1. Verify the connection.

   *  Open an SSH connection to your EC2 instance
   *  On the EC2 instance, install the Redis client:

      ```bash
      sudo apt-get install redis
      ```

   *  Add an inbound rule to the EC2 security group: Type - Custom TCP, port - 6379, Source - 0.0.0.0/0
   *  Add an inbound rule to the ElastiCache Cluster security group: Type - Custom TCP, port - 6379, Source - 0.0.0.0/0
   *  Connect to the Redis CLI:

      ```bash
      redis-cli -h <ElastiCache Primary Endpoint host> -p <ElastiCache Primary Endpoint port>
      ```

### Configure Magento to use the cluster

Magento supports multiple types of caching configurations. Generally, the caching configurations are split between frontend and backend. Frontend caching is classified as 'default', used for any cache type. You can customize or split into lower-level caches to achieve better performance. A common Redis configuration is separating the default cache and page cache into their own Redis Database (RDB).

Run `setup` commands to specify the Redis endpoints.

Configuring Magento for Redis as default caching:

```bash
bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=<ElastiCache Primary Endpoint host>  --cache-backend-redis-port=<ElastiCache Primary Endpoint port> --cache-backend-redis-db=0
```

Configure Magento for Redis page caching:

```bash
bin/magento setup:config:set --page-cache=redis --page-cache-redis-server=<ElastiCache Primary Endpoint host> --page-cache-redis-port=<ElastiCache Primary Endpoint port> --page-cache-redis-db=1
```

Configure Magento to use Redis for session storage:

```bash
bin/magento setup:config:set --session-save=redis --session-save-redis-host=<ElastiCache Primary Endpoint host> --session-save-redis-port=<ElastiCache Primary Endpoint port> --session-save-redis-log-level=4 --session-save-redis-db=2
```

### Verify connectivity

To verify that Magento is talking to ElasiCache:

1. Open an SSH connection to the Magento EC2 instance.
1. Start the Redis monitor.

   ```bash
   redis-cli -h <ElastiCache Primary Endpoint host> -p <ElastiCache Primary Endpoint port> monitor
   ```

1. Open a page in the Magento UI.
1. Verify the [cache output]({{ page.baseurl }}/config-guide/redis/redis-pg-cache.html#redis-verify) in your terminal.

## New Redis cache implementation

As of Magento 2.3.5, it is recommended to use the extended Redis cache implementation: `\Magento\Framework\Cache\Backend\Redis`.

```php
'cache' => [
    'frontend' => [
        'default' => [
            'backend' => '\\Magento\\Framework\\Cache\\Backend\\Redis',
            'backend_options' => [
                'server' => '127.0.0.1',
                'database' => '0',
                'port' => '6379'
            ],
        ],
],
```

## Redis preload feature

Since Magento stores a lot of configuration data in the Redis cache, we can preload data that is reused between pages.
To find keys that need to be preloaded, you need to analyze data that is transferred from Redis to Magento. We suggest to preload data that is loaded on every page, common examples are `SYSTEM_DEFAULT`, `EAV_ENTITY_TYPES`, `DB_IS_UP_TO_DATE`.
Redis uses the `pipeline` in order to composite load requests.
Please note that keys should include the database prefix, e.g. if database prefix is `061_`, preload key will looks like `061_SYSTEM_DEFAULT`.

```php
'cache' => [
    'frontend' => [
        'default' => [
            'id_prefix' => '061_',
            'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
            'backend_options' => [
                'server' => 'redis',
                'database' => '0',
                'port' => '6379',
                'password' => '',
                'compress_data' => '1',
                'compression_lib' => '',
                'preload_keys' => [
                    '061_EAV_ENTITY_TYPES',
                    '061_GLOBAL_PLUGIN_LIST',
                    '061_DB_IS_UP_TO_DATE',
                    '061_SYSTEM_DEFAULT',
                ],
            ]
        ],
        'page_cache' => [
            'id_prefix' => '061_'
        ]
    ]
]
```

In case you are using the preload feature with the L2 cache, do not forget to add the ':hash' suffix to your keys, since L2 cache only transfers the hash of the data, not the data itself:

```php
'preload_keys' => [
    '061_EAV_ENTITY_TYPES:hash',
    '061_GLOBAL_PLUGIN_LIST:hash',
    '061_DB_IS_UP_TO_DATE:hash',
    '061_SYSTEM_DEFAULT:hash',
],
```

## Parallel generation

Starting with the 2.4.0 release, we introduced the `allow_parallel_generation` option for the users that want to eliminate waitings for locks.
It is disabled by default, and we recommend keeping it disabled until you have excessive configurations and/or blocks.

To enable it, run:

```bash
bin/magento setup:config:set --allow-parallel-generation
```

Since it is a flag, you cannot disable it with a command. You will need to manually set the configuration value to `false`:

```php
    'cache' => [
        'frontend' => [
            'default' => [
                'id_prefix' => 'b0b_',
                'backend' => 'Magento\\Framework\\Cache\\Backend\\Redis',
                'backend_options' => [
                    'server' => 'redis',
                    'database' => '0',
                    'port' => '6379',
                    'password' => '',
                    'compress_data' => '1',
                    'compression_lib' => ''
                ]
            ],
            'page_cache' => [
                'id_prefix' => 'b0b_'
            ]
        ],
        'allow_parallel_generation' => false
    ],
```

## Basic verification {#redis-verify}

{% include config/redis-verify.md %}

*  [Use Redis for session storage]({{ page.baseurl }}/config-guide/redis/redis-session.html)
*  [Create or extend configuration types]({{ page.baseurl }}/config-guide/config/config-create.html)
*  [Magento's deployment configuration]({{ page.baseurl }}/config-guide/config/config-php.html)
