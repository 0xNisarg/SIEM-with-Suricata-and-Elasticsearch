# SIEM WITH SURICATA AND ELK STACK

## SETTING UP THE SURICATA ON UBUNTU SERVER 

## Step 1 — Installing Suricata
Run the following command to add the repository to your system and update the list of available packages:
```bash
sudo add-apt-repository ppa:oisf/suricata-stable
```
Now you can install the suricata package using the apt command:
```bash
sudo apt install suricata
```
Now that the package is installed, enable the suricata.service so that it will run when your system restarts. Use the systemctl command to enable it:
```bash
sudo systemctl enable suricata.service
```
Before moving on to the next section of this tutorial, which explains how to configure Suricata, stop the service using systemctl:
```bash
sudo systemctl stop suricata.service
```

## Step 2 — Configuring Suricata For The First Time
If you plan to use Suricata with other tools like Zeek or Elasticsearch, adding the Community ID now is a good idea.

To enable the option, open /etc/suricata/suricata.yaml using nano or your preferred editor:
```bash
sudo nano /etc/suricata/suricata.yaml
```
Find line 120 which reads # Community Flow ID. Below that line is the community-id key. Set it to true to enable the setting
```bash
 # Community Flow ID
      # Adds a 'community_id' field to EVE records. These are meant to give
      # records a predictable flow ID that can be used to match records to
      # output of other tools such as Zeek (Bro).
      #
      # Takes a 'seed' that needs to be same across sensors and tools
      # to make the id less predictable.

      # enable/disable the community id feature.
      community-id: true
```
Now when you examine events, they will have an ID like 1:S+3BA2UmrHK0Pk+u3XH78GAFTtQ= that you can use to correlate records across different NMS tools.
Save and close the /etc/suricata/suricata.yaml file.

### Determining Which Network Interface(s) To Use

To determine the device name of your default network interface, you can use the ip command as follows:
```bash
ip -p -j route show default
```
You should receive output like the following:
```bash
[ {
        "dst": "default",
        "gateway": "203.0.113.254",
        "dev": "eth0",
        "protocol": "static",
        "flags": [ ]
    } ]
```
Now you can edit Suricata’s configuration and verify or change the interface name. Open the /etc/suricata/suricata.yaml configuration file using nano or your preferred editor:
```bash
sudo nano /etc/suricata/suricata.yaml
```
```bash
# Linux high speed capture support
af-packet:
  - interface: eth0
    # Number of receive threads. "auto" uses the number of cores
    #threads: auto
    # Default clusterid. AF_PACKET will load balance packets based on flow.
    cluster-id: 99
```
### Configuring Live Rule Reloading
Suricata supports live rule reloading, which means you can add, remove, and edit rules without needing to restart the running Suricata process. To enable the live reload option, scroll to the bottom of the configuration file and add the following lines:
```bash
detect-engine:
  - rule-reload: true
```
Save and close the /etc/suricata/suricata.yaml file.

## Step 3 — Updating Suricata Rulesets
Suricata includes a tool called suricata-update that can fetch rulesets from external providers. Run it as follows to download an up to date ruleset for your Suricata server:
```bash
sudo suricata-update
```
You should receive output like the following:
```bash
19/10/2021 -- 19:31:03 - <Info> -- Using data-directory /var/lib/suricata.
19/10/2021 -- 19:31:03 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
19/10/2021 -- 19:31:03 - <Info> -- Using /etc/suricata/rules for Suricata provided rules.
. . .
19/10/2021 -- 19:31:03 - <Info> -- No sources configured, will use Emerging Threats Open
19/10/2021 -- 19:31:03 - <Info> -- Fetching https://rules.emergingthreats.net/open/suricata-6.0.3/emerging.rules.tar.gz.
 100% - 3044855/3044855               
. . .
19/10/2021 -- 19:31:06 - <Info> -- Writing rules to /var/lib/suricata/rules/suricata.rules: total: 31011; enabled: 23649; added: 31011; removed 0; modified: 0
19/10/2021 -- 19:31:07 - <Info> -- Writing /var/lib/suricata/rules/classification.config
19/10/2021 -- 19:31:07 - <Info> -- Testing with suricata -T.
19/10/2021 -- 19:31:32 - <Info> -- Done.
```

### Adding Ruleset Providers
You can list the default set of rule providers using the list-sources flag to suricata-update like this:
```bash
sudo suricata-update list-sources
```
You will receive a list of sources like the following:
```bash
19/10/2021 -- 19:27:34 - <Info> -- Adding all sources
19/10/2021 -- 19:27:34 - <Info> -- Saved /var/lib/suricata/update/cache/index.yaml
Name: et/open
  Vendor: Proofpoint
  Summary: Emerging Threats Open Ruleset
  License: MIT
```

## Step 4 — Validating Suricata’s Configuration
For Test the configuration Run The following command :
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
With the default ET Open ruleset you should receive output like the following:
```bash
Output
21/10/2021 -- 15:00:40 - <Info> - Running suricata under test mode
21/10/2021 -- 15:00:40 - <Notice> - This is Suricata version 6.0.3 RELEASE running in SYSTEM mode
21/10/2021 -- 15:00:40 - <Info> - CPUs/cores online: 2
21/10/2021 -- 15:00:40 - <Info> - fast output device (regular) initialized: fast.log
21/10/2021 -- 15:00:40 - <Info> - eve-log output device (regular) initialized: eve.json
21/10/2021 -- 15:00:40 - <Info> - stats output device (regular) initialized: stats.log
21/10/2021 -- 15:00:46 - <Info> - 1 rule files processed. 23879 rules successfully loaded, 0 rules failed
21/10/2021 -- 15:00:46 - <Info> - Threshold config parsed: 0 rule(s) found
21/10/2021 -- 15:00:47 - <Info> - 23882 signatures processed. 1183 are IP-only rules, 4043 are inspecting packet payload, 18453 inspect application layer, 107 are decoder event only
21/10/2021 -- 15:01:13 - <Notice> - Configuration provided was successfully loaded. Exiting.
21/10/2021 -- 15:01:13 - <Info> - cleaning up signature grouping structure... complete
```

## Step 5 — Running Suricata
Now that you have a valid Suricata configuration and ruleset, you can start the Suricata server. Run the following systemctl command:
```bash
sudo systemctl start suricata.service
```
You can examine the status of the service using the systemctl status command:
```bash
sudo systemctl status suricata.service
```
You should receive output like the following:
```bash
Output
● suricata.service - LSB: Next Generation IDS/IPS
     Loaded: loaded (/etc/init.d/suricata; generated)
     Active: active (running) since Thu 2021-10-21 18:22:56 UTC; 1min 57s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 22636 ExecStart=/etc/init.d/suricata start (code=exited, status=0/SUCCESS)
      Tasks: 8 (limit: 2344)
     Memory: 359.2M
     CGroup: /system.slice/suricata.service
             └─22656 /usr/bin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid --af-packet -D -vvv

Oct 21 18:22:56 suricata systemd[1]: Starting LSB: Next Generation IDS/IPS...
Oct 21 18:22:56 suricata suricata[22636]: Starting suricata in IDS (af-packet) mode... done.
Oct 21 18:22:56 suricata systemd[1]: Started LSB: Next Generation IDS/IPS.
```
As with the test mode command, it will take Suricata a minute or two to load and parse all of the rules. You can use the tail command to watch for a specific message in Suricata’s logs that indicates it has finished starting:
```bash
sudo tail -f /var/log/suricata/suricata.log
```
