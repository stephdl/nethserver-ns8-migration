========================
nethserver-ns8-migration
========================

This package will migrate services from NethServer 7 to NethServer 8 (NS8).

On install the ``nethserver-ns8-migration-update`` event creates a WireGuard private key.
This key is later used to join NS8 cluster as a special node which *can't* be used to run containers.

The ``ns8-join`` script can be used to retrieve Wireguard configuration: ::

  ns8-join [--no-tlsverify] <ns8_host> <admin_user> <admin_pass>

After executing the ``ns8-join`` command, the ``nethserver-ns8-migration-save`` event will establish
the VPN connection.

Example: ::

  ns8-join ns8.nethserver.org admin Nethesis,1234
  signal-event nethserver-ns8-migration-save


After the join, it is possible to execute actions: ::

  ns8-action <agent> <action> <json_data>

Example: ::

  ns8-action cluster get-cluster-status '{}'

Ldapproxy
=========

The ``/usr/share/nethesis/nethserver-ns8-migration/apps/ldapproxy/export`` script configures NS8 ldapproxy to connect
to the local Samba Active Directory running on NS7.

The script will:
- create a static route inside nsdc container
- update NS8 VPN routes to connect local green interface
- configure ldapproxy for connection to Samba AD using WireGuard VPN

Assumptions:
- the AD has an IP address of the first green network interface

Nextcloud migration
===================

The migration only supports Nextcloud configured with a virtual host inside NS7.
At the end, the migrated Nextcloud install will be accessible at the same virtual host on the remote NS8 cluster.

Assumptions:
- nextcloud is connected to the local Samba AD

Steps:

1. After the ns8-join, configure ldapproxy to connect to local Samba AD: ::

      /usr/share/nethesis/nethserver-ns8-migration/apps/ldapproxy/export

2. The first time, install a Nextcloud instance to NS8. Then synchronize data, configuration and database: ::

     /usr/share/nethesis/nethserver-ns8-migration/apps/nextcloud/export

   This command can be executed multiple times to sync the data until Nextcloud is ready to be totally migrated.

3. When ready for the final migration, execute: ::

     /usr/share/nethesis/nethserver-ns8-migration/apps/nextcloud/migrate
