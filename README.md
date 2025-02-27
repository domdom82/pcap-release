# pcap-release

[BOSH](https://bosh.io/) release of the pcap [Cloud Foundry](https://www.cloudfoundry.org/) add-on.

## Disclaimer

As pcap-release is still in active development, information on this page is subject to change and might be outdated.

## Description

The goal of this BOSH release is to provide easy access to network traffic data to both Cloud Foundry application developers and CF landscape operators. To achieve this, pcap-release implements CLI-commands to capture tcpdumps (pcap files) from multiple BOSH VMs or CF app-containers in parallel.

<!-- TODO: to be added later
For the BOSH VM capture case, a new CLI can be used that authenticates via the BOSH director.
For tcpdumps of CF app containers, pcap-release provides a plugin to the CF Cloud Controller CLI.
-->

## Architecture

### CF App Capture

* `pcap-api` is deployed on its own VM.
* `pcap-agent` is co-located on app-containers.
* `pcap-api` needs to register its route via route-registrar, part of the cf-routing-release.
* `cf-CLI` pcap-plugin (TBD/WIP) is used to send capture requests to the `pcap-api`
* Requests to `pcap-api` need an authorization header including the OAuth token from UAA.
  This token is used to gather information about the app from the cloud-controller.
* The `pcap-api` makes requests to the `pcap-agent` on corresponding app-container.
* The pcap agent starts a tcpdump using libpcap via [the gopacket module](https://github.com/google/gopacket) and streams the results.

![tcpdump in cf architecture](docs/tcpdump-for-cf.svg "tcpdump in cf architecture")

## BOSH VM Capture

* `pcap-api` is deployed on its own VM.
* `pcap-agent` is co-located on BOSH VMs.
* `pcap-api` needs to register its route via route-registrar, part of the cf-routing-release.
* `pcap-bosh-cli` (TBD/WIP) is used to send capture requests to the `pcap-api`
* Requests to `pcap-api` need an authorization header including the OAuth token from UAA.
* This token is used to gather information about the target VMs from the BOSH Director
* The `pcap-api` makes requests to the `pcap-agent` on corresponding BOSH VMs.
* The pcap agent starts a tcpdump using libpcap via [the gopacket module](https://github.com/google/gopacket) and streams the results.

![tcpdump in bosh architecture](docs/tcpdump-for-bosh.svg "tcpdump in bosh architecture")

## Jobs

### pcap-api

* Check if token is valid by requesting app information from CC/BOSH Director
* Get container address for app instances from CC/BOSH VMs from BOSH Director
* Connect to `pcap-agent` in app-containers/BOSH VMs
* Stream packets back to client

### pcap-agent

* Capture packets and stream back to `pcap-api`

## How to deploy

The release provides two files to integrate with an
existing [cf-deployment](https://github.com/cloudfoundry/cf-deployment):

* `manifests/ops-files/add-pcap-agent.yml` This provides a shared CA between pcap-agent and pcap-api. It also adds the pcap-agent job to all diego cells.
* `manifests/pcap-api.yml` This is an example BOSH manifest to deploy the pcap-api

### Step 1 - Add pcap-agent to cf-deployment

```bash
bosh interpolate -o manifests/ops-files/add-pcap-agent.yml cf-deployment.yml > cf-deployment-pcap.yml
bosh -d cf deploy cf-deployment-pcap.yml
```

This assumes your BOSH deployment name of cf-deployment is called `cf`

### Step 2 - Deploy pcap-api

```bash
cp manifests/vars-template.yml manifests/vars.yml
vim manifests/vars.yml (adjust as needed)
bosh -d pcap deploy -l manifests/vars.yml manifests/pcap-api.yml
```

### Step 3 - Install CF CLI plugin

```bash
wget https://pcap.cf.cfapp.com/cli/pcap-cli-[linux|mac]-amd64 (adjust URL as needed) -O pcap-cli
cf install-plugin pcap-cli
cf pcap ...
```

## Development Deployment for BOSH

```shell
# create a dev release, event if there are changes in the git workspace
bosh create-release --force

# upload the release to the BOSH director
bosh -e bosh upload-release

# adjust the release to a dev release instead of the URL
vim manifests/pcap-api.yml
vim manifests/ops-files/add-pcap-agent-haproxy.yml

# deploy pcap-agent to the HAProxy deployment(s)
bosh interpolate -o manifests/ops-files/add-pcap-agent.yml haproxy.yml > haproxy-pcap.yml
bosh -d cf haproxy haproxy-pcap.yml

# deploy pcap-api
cp manifests/vars-template.yml manifests/vars.yml
vim manifests/vars.yml (adjust as needed)
bosh -d pcap deploy -l manifests/vars.yml manifests/pcap-api.yml
```
