---
description: How to add adapools.org summary.json info to your Grafana instance.
---

# Add adapools Metrics to Grafana 📊

## Assumptions

You have set up a Cardano node using one of the tutorials provided [here](pi-pool-tutorial/). If so, you should have the necessary dependencies installed that the steps below utilize. If not, see the apt install [Environment Setup](../cardano-node/raspi-node/environment-setup.md#install-packages) section of the Pi-Pool Tutorial.

## Make New Directory

To start, pick a location on the machine that is running Grafana where you will create a new directory for the node exporter to use. The node exporter is likely located in /opt/cardano/monitoring/**node\_exporter** given the pi-pool default location. To start, pick a location on the machine that is running Grafana where you will create a new directory for the node exporter to use. If that doesn't find it, the directory where it's located is not on your $PATH and you'll need to dig deeper. [Check this git](https://github.com/prometheus/node_exporter) for more information.

Change to the location for the new directory, here I'm selecting the local bin for my user.

```text
> cd $HOME/.local/bin
```

Now make a new directory here where we can store custom text file stats that the node\_exporter will parse. I'm calling the directory **customStats**, but you can name it whatever you like.

```text
> mkdir customStats
```

## Get adapools Summary File

The adapools.org site provides a **summary.json** file for every registered pool. We'll use this file to parse out the data we want and store it in our directory we just created. We can create a bash script to handle this for us. I'm in my $HOME/.local/bin directory:

```text
> nano getAdaPoolsSummary.sh
```

Add this content below, replace **YOUR POOL ID** with your pool's ID, save and exit. Essentially this pulls a copy of the **summary.json** file for your pool, removes some things that the node exporter cannot parse \(string values\) and saves a copy in our new directory.

```text
curl https://js.adapools.org/pools/<YOUR POOL ID>/summary.json 2>/dev/null \
| jq '.data | del(.direct, .hist_bpe, .handles, .hist_roa, .db_ticker, .db_name, .db_description, .db_url, .ticker_orig, .pool_id, .pool_id_bech32, .group_basic)' \
| tr -d \"{},: \
| awk NF \
| sed -e 's/^[ \t]*/adapools_/' > $HOME/.local/bin/customStats/adapools.prom
```

Now when the **getAdaPoolsSummary.sh** is run it'll refresh a file called **adapools.prom** in our new directory. This file will contain metrics that start with the term **adapools** and will be visible in the Grafana query builder metrics section as such.

{% hint style="warning" %}
It's important that the results in the file do not include string values. The node exporter will throw an error and you won't see the adapools metrics.
{% endhint %}

If you discover string values, you can remove them by adding a new key to the "del" section in the script above. For example, to remove the **adapools\_db\_description** metric \(has a string value\), you'd add **.db\_description** to the **del\( \)** section.

## Create crontab Entry

Depending on how often you want to refresh a copy of these stats, you can create a local crontab entry to pull a fresh copy of the adapools.prom file.

```text
> crontab -e
```

The following line **runs the script we created every 5 minutes**. Add the line, save and exit. Since this data doesn't change that often, you shouldn't need to pull it that often. Don't piss off the adapools.org folks by pulling this data every 5 seconds - it's not necessary. For other examples of crontab run times, [see this lovely link](https://crontab.tech/examples).

```text
*/5 * * * * $HOME/.local/bin/getAdaPoolsSummary.sh
```

## Run node exporter Command

Now that we are generating the **adapools.prom** file, we need to tell the node exporter where to find our custom text file. Depending on how you are running your node exporter instance, you'll need to add the following command line parameters. This might be found in the **startMonitor** script included with the pi-pool default build.

```text
> node_exporter --collector.textfile.directory=$HOME/.local/bin/customStats --collector.textfile
```

If all goes as planned, you should be able to pull up this URL in your browser and see the new **adapools** metrics. If this worked, your new metrics should be visible in the Grafana query builder.

```text
http://<YOUR GRAFANA NODE IP>:9100/metrics
```

{% hint style="info" %}
There are other methods you could use to implement this approach. Basically, if you can create a text file with key/value pairs and put it into this new directory, the node exporter should pull the data into Grafana. It opens up a vast array of possibilities. Just ensure you prefix the label names with a unique value \(the **adapools\_** \_\_part in the adapools.prom file above\) per file.
{% endhint %}

Was this information helpful? Earn rewards with us! [Consider delegating some ADA](../cardano-developer-guides/delegate.md).

