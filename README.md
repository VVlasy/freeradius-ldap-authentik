Introduction
============

FreeRadius server configured to use an Authentik LDAP provider. Can be used as a UniFi WiFi or VPN Radius authentication backend.

Optional support is provided so that users must be a member of a certain LDAP
group in order to receive RADIUS access. If this is not used, all users which
can authenticate successfully using LDAP will be granted RADIUS access.

Authentik setup
====================================
First an LDAP outpost is necessary, either in docker-compose or managed by Authentik using local docker socket. I recommend creating a service user in authentik.

For a basic radius server, this will get you up and running. For authenticating with MsChapv2 (UniFi VPN and WiFi auth), additional setup is necessary in Authentik:

In the flow `default-password-change` (to adapt if you have a custom one), you'd go to "Stage Bindings", expand the `default-password-change-write` binding, click `Create and bind Policy`, select `Expression Policy`, give it a meaningful name, and use the following [expression provided](https://gist.github.com/jcrapuchettes/830c99982e391c858f5b7eb066c02749). Your users will have to change their password for them to be able to use the output, for other solutions during the auth flow, [see the thread here](https://github.com/goauthentik/authentik/issues/8768#issuecomment-2383738234).

Many thanks to [@jcrapuchettes](https://github.com/jcrapuchettes) who already spent his time and effort.

Custom certificate
====================================
Mount the directory `/etc/raddb/certs/rsa` where you have `server.pem` and `server.key` files with your own certificate. 
Please include a `ca.pem` with your CA public certificate in the same directory.
Set `EAP_PRIVATE_KEY_PASSWORD` to match your private key password.

In case of permission problems, run `chmod 664 *` in the directory where your certificates are stored.

Environment Variables
=====================

- `LDAP_HOST` - LDAP server hostname(s) (default: "ldap1.example.com ldap2.example.com")
- `LDAP_PORT` - LDAP server port (default: "389")
- `LDAP_USER` - LDAP server user (default: "cn=admin,dc=example,dc=com")
- `LDAP_PASS` - LDAP server password (default: "password")
- `LDAP_BASEDN` - LDAP server Base DN (default: "dc=example,dc=com")
- `LDAP_USER_BASEDN` - LDAP server Users Base DN (default: "ou=Users,dc=example,dc=com")
- `LDAP_GROUP_BASEDN` - LDAP server Groups Base DN (default: "ou=Groups,dc=example,dc=com")
- `LDAP_CLIENT_BASEDN` - LDAP server Freeradius Clients Base DN (default: "ou=Clients,dc=example,dc=com")

- `LDAP_RADIUS_ACCESS_GROUP` - The LDAP group which users must belong to in order to have RADIUS access (optional) (default: "")
- `RADIUS_CLIENT_CREDENTIALS` - The Freeradius server client credentials (comma separated "hostname:password" pairs, default: "")

- `EAP_PRIVATE_KEY_PASSWORD` - Password for private key when custom cert is provided in volumes, when using self signed certificate, do not set this variable

- `RADIUSD_ARGS` - Arguments to pass to radiusd (default: "-f -l stdout")

Example Docker Compose Configuration
====================================

    radius:
      image: vvlasy/freeradius-ldap:master
      ports:
        - "1812:1812/udp"
        - "1813:1813/udp"
      volumes:
        - ./custom-cert:/etc/raddb/certs/rsa
      environment:
        #- RADIUSD_ARGS=-x -f -l stdout
        - "LDAP_HOST=ldap.example.com"
        - "LDAP_USER=cn=admin,dc=example,dc=com"
        - "LDAP_PASS=adminpassword"
        - "LDAP_BASEDN=dc=example,dc=com"
        - "LDAP_USER_BASEDN=ou=Users,dc=example,dc=com"
        - "LDAP_GROUP_BASEDN=ou=Groups,dc=example,dc=com"
        - "LDAP_RADIUS_ACCESS_GROUP=vpnaccess"
        - "RADIUS_CLIENT_CREDENTIALS=1.2.3.4:password1234,5.6.7.8:password5678"
        #- EAP_PRIVATE_KEY_PASSWORD=whatever
      mem_limit: "1g"
      restart: "always"
