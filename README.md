# IBP Prometheus Ansible

This project contains the automation of the monitoring stack for IBP RPC via Prometheus metrics, using Ansible roles.
The monitoring instance management is by nature centralized but each member can send a PR to add their nodes to monitor.
Members have 2 choices to connect their nodes to this monitoring:
- provide themselves an authorised access to each node chain metrics (default port 9615)
- connect to the IBP monitoring private network via Wireguard (see below)

The monitoring endpoint is available at https://ibp-monitor.bldnodes.org


## Federated metrics

Members using their own prometheus can setup [federated metrics](https://github.com/ibp-network/member-prometheus) to avoid exposing their node metric endpoints.

## Add nodes to Prometheus

Update `roles/prometheus/files/prometheus.yml` by adding one target for each node. Add a `job_name` for a new member.
You need to add the following labels for each target: `member`, `chain` and `node_type`. 

Please respect the naming convention in place for label in low caps.

`chain` values:
- polkadot
- kusama
- westend
- statemint
- statemine
- westmint
- collectives-polkadot
- collectives-westend
- bridge-hub-polkadot
- bridge-hub-kusama
- bridge-hub-westend
- encointer


## Connect server via the IBP monitoring private network

Pre-requisite: ask for a VPN IP address in the group `IBP Members`.

Install Wireguard

        sudo apt-get install wireguard

Generate the VPN private and public keys:

        umask 077
        ### {SERVER_PRIVATE_KEY} ###
        wg genkey | sudo tee /etc/wireguard/private.key
        ### {SERVER_PUBLIC_KEY} ###
        sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key

Create the Wireguard config file in `/etc/wireguard/wg0.conf`

        [Interface]
        PrivateKey = {SERVER_PRIVATE_KEY}
        Address = {VPN_IP_ASSIGNED}/32
        ListenPort = 35760
        [Peer]
        PublicKey = sEkLBxeo32o8wm4Pk+bdu6ePVj1gwxDKBQ/CUGf0fx8=
        AllowedIPs = 172.23.0.1/32
        Endpoint = ibp-monitor.bldnodes.org:35759
        PersistentKeepalive = 60

If you are using ufw firewall, you need to allow the interface wg0 and the listen port:

        sudo ufw allow in on wg0 to any
        sudo ufw allow 51100

Start the service:

        sudo systemctl start wg-quick@wg0

Verify the VPN connectivity

        sudo wg

Once your are set, create a PR with the update of Prometheus file and add `{SERVER_PUBLIC_KEY}` and `{VPN_IP_ASSIGNED}` in comment of the PR.
Note: make sure to send only your public key, NOT YOUR PRIVATE KEY!
