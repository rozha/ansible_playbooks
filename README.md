# What is this?

This is a collection of Ansible roles I created to run my own automations for living.

# Available Roles

## digital_ocean_droplet

**Ansible >= 2.8 is required.**

Don't forget to set the `DO_API_TOKEN' environment variable before firing up Ansible for this.

If you wish to use SSH keys, which you probably do, you have to set the `digital_ocean_droplet_ssh_keys` Ansible variable to the array containing ids of your ssh key[s].  For example:

`ansible_playbook digital_ocean_openvpn.yml -e 'digital_ocean_droplet_ssh_keys="[12345]"'`

Where **12345** is the identifier of your SSH key.  You can get ids of the SSH keys you uploaded to Digital Ocean using the following command

`curl -s -X GET -H "Content-Type: application/json" -H "Authorization: Bearer API_TOKEN" "https://api.digitalocean.com/v2/account/keys?page=1&per_page=100" | python -m json.tool`

Make sure to replace **API_TOKEN** with your actual DO API token.  You can also append `| grep -E '"(id|name)"'` to the command provided above to limit the output to ids and names only.  There are other params such as image name, region, and vm size which can accept only predefined values.  To list those values, refer to https://developers.digitalocean.com/documentation/v2/.  Another option is to use [doctl](https://github.com/digitalocean/doctl "doctl github page"), e.g.

`doctl compute ssh-key ls`

`doctl compute image list-distribution`

`doctl compute region ls`

### Supported variables and their default values

| Variable | Default | Description |
| -------- | ------- | ----------- |
digital_ocean_droplet_name         | mydroplet | Name which will be used for your brand-new droplet
digital_ocean_droplet_size         | s-1vcpu-1gb | Size of your droplet<br/>`doctl compute size ls`
digital_ocean_droplet_image        | debian-10-x64 | Digital Ocean image used for droplet creation<br/>`doctl compute image list-distribution`
digital_ocean_droplet_region       | sgp1 | Digital Ocean data center to run the droplet in<br/>`doctl compute region ls`
digital_ocean_droplet_ipv6         | no | IPv6 support
digital_ocean_droplet_backups      | no | Enable backups
digital_ocean_droplet_monitoring   | no | Enable additional monitoring
digital_ocean_droplet_unique_name  | no | Enable unique names
digital_ocean_droplet_private_networking | no | Enable private networking
digital_ocean_droplet_tags         | [] | Tags the droplet will be tagged with
digital_ocean_droplet_volumes      | [] | Additional volumes to attach to the droplet
digital_ocean_droplet_ssh_keys     | [] | SSH keys to add to the root user's authorized_keys<br/>`doctl compute ssh-key ls`
digital_ocean_droplet_user_data    | | User data to initialise the droplet with
digital_ocean_droplet_wait         | yes | Wait for droplet creation
digital_ocean_droplet_wait_timeout | 600 | Max number of settings to wait for droplet upon its creation

## openvpn\_server

**Ansible >= 2.4 is required.**

**netaddr Python module is required, `pip install netaddr`**

This role installs and configures OpenVPN server.  It also installs and configures [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page "nftables wiki") if `/etc/nftables.conf` doesn't exist.

So far, I've tested this role on Digital Ocean only.  Other cloud providers may or may not be tested in the future but should work with this role anyway.  It was tested with Debian 10 only for now.

This role doesn't rely on the version of EasyRSA shipped by your operating system.  Instead, it downloads EasyRSA distro from Github.  Also, upon successful completion this role downloads OpenVPN client configuration file to the **openvpn_server_config_download_path** directory which is **openvpn_conf** by default.  That configuration file contains all required key and cert data inlined.

### Supported variables and their default values

| Variable | Default | Description |
| -------- | ------- | ----------- |
openvpn_server_port         | 1488 | Port OpenVPN server will be listening on
openvpn_server_protocol     | udp | Protocol to be used for VPN connections, udp or tcp
openvpn_server_dev          | tun | Driver to be used for VPN connections
openvpn_server_mtu          | 1400 | MTU
openvpn_server_openvpn_dir  | /etc/openvpn | Directory to store OpenVPN configuration files
openvpn_server_log_dir      | /var/log/openvpn | Directory to store OpenVPN log data
openvpn_server_easyrsa_dir  | /etc/easy-rsa | Directory to use for easyrsa data
openvpn_server_easyrsa_dist_url | [EasyRSA-unix-v3.0.6.tgz](https://github.com/OpenVPN/easy-rsa/releases/download/v3.0.6/EasyRSA-unix-v3.0.6.tgz "download easyrsa from Github") | Easyrsa distro
openvpn_server_easyrsa_dist_checksum | [SHA256 hash here](#sha256:cb29aed2d27824e59dbaad547f11dcab380a53c9fe05681249e804af436f1396 "sha256:cb29aed2d27824e59dbaad547f11dcab380a53c9fe05681249e804af436f1396") | Easyrsa distro checksum
openvpn_server_dh           | {{ openvpn_server_easyrsa_dir }}/keys/dh4096.pem | Location of Diffie–Hellman pem
openvpn_server_dh_bits      | 2048 | Size of Diffie–Hellman parameters
openvpn_server_ca_cert      | {{ openvpn_server_easyrsa_dir }}/keys/ca.crt | Location of CA certificate
openvpn_server_ca_key       | {{ openvpn_server_easyrsa_dir }}/keys/ca.key | Location of CA key
openvpn_server_ca_cert_days | 3652 | Lenght of validity of CA certificate
openvpn_server_pki_dir      | {{ openvpn_server_easyrsa_dir }}/pki | Directory containing PKI data
openvpn_server_csr          | {{ openvpn_server_easyrsa_dir }}/keys/server.csr | Location of OpenVPN server CSR
openvpn_server_cert         | {{ openvpn_server_easyrsa_dir }}/keys/server.crt | Location of OpenVPN server cert
openvpn_server_cert_days    | 3652 | Lenght of validity of server certificate
openvpn_server_key          | {{ openvpn_server_easyrsa_dir }}/keys/server.key | Location of OpenVPN server key
openvpn_server_key_bits     | 4096 | Size of OpenVPN server key
openvpn_server_conf         | {{ openvpn_server_openvpn_dir }}/server.conf | Path to OpenVPN server configuration file
openvpn_server_network      | 172.16.55.0 | IPv4 network to be used for VPN
openvpn_server_mask         | 255.255.255.0 | Netmask of the network specified above
openvpn_server_status_log   | {{ openvpn_server_log_dir }}/server-status.log | OpenVPN server status log ifle
openvpn_server_log          | {{ openvpn_server_log_dir }}/server.log | OpenVPN server log file
openvpn_server_verb         | 3 | OpenVPN server log ffile verbosity
openvpn_server_cipher       | AES-256-GCM | Cipher to be used for VPN connections
openvpn_server_keepalive    | "10 120" | OpenVPN server keepalive settings
openvpn_server_client_csr   | {{ openvpn_server_easyrsa_dir }}/keys/client-{{ ansible_default_ipv4.address }}.csr | File client CSR will be stored in
openvpn_server_client_cert  | {{ openvpn_server_easyrsa_dir }}/keys/client-{{ ansible_default_ipv4.address }}.crt | Where to store client certificate
openvpn_server_client_cert_days | 3652 | Lenght of validity of client certificate
openvpn_server_client_key   | {{ openvpn_server_easyrsa_dir }}/keys/client-{{ ansible_default_ipv4.address }}.key | File to store client key data
openvpn_server_client_key_bits | 4096 | Size of OpenVPN client key
openvpn_server_client_conf  | {{ openvpn_server_openvpn_dir }}/client-{{ ansible_default_ipv4.address }}.conf | Path OpenVPN client config will be stored in
openvpn_server_nftables_config_file | /etc/nftables.conf | Path to nftables.conf
openvpn_server_config_download_path | openvpn_conf/ | Local directory to fetch OpenVPN client configuration file to; if you want it it to be a file and overwritten at each run, remove the trailing slash and, optionally, append an extension.
openvpn_server_enable_tls_auth | yes | Enable tls-auth settings
openvpn_server_tls_auth_key | {{ openvpn_server_openvpn_dir }}/tls-auth.key | Path to store tls-auth key data
openvpn_server_enable_management | yes | Enable OpenVPN server management interface
openvpn_server_management_port | 9999 | OpenVPN server management interface port
openvpn_server_client_enable_management | no | Enable OpenVPN client management interface
openvpn_server_client_management_port | 9999 | OpenVPN client management interface port

-----

# Information below is outdated and left here for reference purposes only

## ec2\_proftpd.yml

* ensure an 'ftpd' security group exists (TCP 20-22, 25000-27000)
* create a t1.micro instance
* update system using the package manager
* install, configure, and run ProFTPD and fail2ban

Tested with the following AMIs:

* ami-98aa1cf0 Ubuntu Server 14.04 LTS (PV), SSD Volume Type, 64 bit
* ami-246ed34c Amazon Linux AMI 2014.09.1 (PV), 64 bit
* ami-1643ff7e Red Hat Enterprise Linux 6.5 (PV), 64 bit

Usage:

* edit **vars/ec2\_proftpd**
 * **image**: AMI to create your new instance from
 * **instance\_type**: type of the instance
 * **keypair**: key pair to use
 * **region**: region to create the new instance in
 * **vpc\_id**: id of the VPC to run the instance in
 * **zone**: AWS availability zone
 * **ssh\_user**: user name to use to log in to the instance via SSH

* invoke the following commands:
 * export AWS\_ACCESS\_KEY=**YOURIAMACCESSKEY**
 * export AWS\_SECRET\_KEY=**YOURIAMSECRETKEY**
 * ansible-playbook ec2\_proftpd.yml

## openvpn\_server.yml

Configure and run an OpenVPN server.  Tested on Debian 8, Ubuntu 12.04, and CentOS 7.
