---
description: General deployment instructions for a hardware or cloud services environment
---

# Manual Deployment

{% hint style="info" %}
For automated deployment on AWS, see [Ansible deployment](../ansible-deployment/).
{% endhint %}

Check your environment is prepared with General Requirements and [Database Storage Requirements](../information-and-settings/database-storage-requirements.md).

BlockScout requires a **full archive node** in order to import every state change for every address on the target network. For client specific settings related to a node running parity or geth, please see [Client Settings](../information-and-settings/client-settings-parity-geth-ganache.md).

{% hint style="info" %}
For testing purposes, instead of an archive node, a test Ethereum client can be used. For instance, [ganache-cli](https://github.com/trufflesuite/ganache-cli)
{% endhint %}

## Deployment Steps

1\) `git clone https://github.com/poanetwork/blockscout`

2\) `cd blockscout`

3\) Provide DB URL:\
`export DATABASE_URL=postgresql://user:password@localhost:5432/blockscout`&#x20;

* **Linux:** Update the database username and password configuration
* **Mac:** Use logged-in user name and empty password
* **Optional:** Change credentials in `apps/explorer/config/test.exs` for test env

{% hint style="info" %}
_Example usage:_ Changing the default Postgres port from localhost:5432 if [Boxen](https://github.com/boxen/boxen) is installed.
{% endhint %}

4\) Install Mix dependencies and compile them \``mix do deps.get, local.rebar --force, deps.compile`\`

5\) Generate a new secret\_key\_base for the DB by setting a corresponding ENV var:\
`export SECRET_KEY_BASE=VTIB3uHDNbvrY0+60ZWgUoUBKDn9ppLR8MI4CpRz4/qLyEFs54ktJfaNT6Z221No`

{% hint style="info" %}
In order to generate a new `secret_key_base` run `mix phx.gen.secret`
{% endhint %}

6\) If you have deployed previously, remove static assets from the previous build `mix phx.digest.clean`.

7\) Set other environment variables as needed.

CLI Example:

```
export ETHEREUM_JSONRPC_VARIANT=parity
export ETHEREUM_JSONRPC_HTTP_URL=http://localhost:8545
export DATABASE_URL=postgresql://...
export COIN=DAI
export ... 
```

{% hint style="info" %}
The `ETHEREUM_JSONRPC_VARIANT` will vary depending on your client (parity, geth etc). [More information on client settings](../information-and-settings/client-settings-parity-geth-ganache.md).
{% endhint %}

8\)  Compile the application:`mix compile`

9\) If not already running, start Postgres: `pg_ctl -D /usr/local/var/postgres start`

{% hint style="success" %}
To check [postgres status](https://www.postgresql.org/docs/9.6/app-pg-isready.html): `pg_isready`
{% endhint %}

10\) Create and migrate database `mix do ecto.create, ecto.migrate`

{% hint style="danger" %}
If you in dev environment and have run the application previously with the different blockchain, drop the previous database `mix do ecto.drop, ecto.create, ecto.migrate`\
Be careful since it will delete all data from the DB. Don't execute it on production if you don't want to lose all the data!
{% endhint %}

11\) Install Node.js dependencies

* `cd apps/block_scout_web/assets; npm ci && node_modules/webpack/bin/webpack.js --mode production; cd -`
* `cd apps/explorer && npm ci; cd -`

12\) Build static assets for deployment `mix phx.digest`

13\) Enable HTTPS in development. The Phoenix server only runs with HTTPS.

* `cd apps/block_scout_web; mix phx.gen.cert blockscout blockscout.local; cd -`
* Add blockscout and blockscout.local to your `/etc/hosts`

```
   127.0.0.1       localhost blockscout blockscout.local

   255.255.255.255 broadcasthost

   ::1             localhost blockscout blockscout.local
```

{% hint style="info" %}
If using Chrome, Enable `chrome://flags/#allow-insecure-localhost`
{% endhint %}

14\) Return to the root directory and start the Phoenix Server. `mix phx.server`

##
