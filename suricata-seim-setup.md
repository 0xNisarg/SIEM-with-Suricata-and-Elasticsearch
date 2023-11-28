# SIEM WITH SURICATA AND ELK STACK

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
