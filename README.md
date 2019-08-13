# VyOS config sync

This replicates the functionality of the commercial version of Vyatta's config-sync for VyOS and community Vyatta editions.

## Work in progress

Now you can get original config-sync from https://github.com/keshavdv/vyattta-config-sync,
install it and copy
https://raw.githubusercontent.com/Harliff/vyattta-config-sync/master/scripts/vyatta-config-sync-update.sh 
over /opt/vyatta/sbin/vyatta-config-sync-update.sh

Maybe I'll create install package some day. 

#### Installation

    $ curl https://github.com/keshavdv/vyattta-config-sync/releases/download/v0.0.1/vyatta-config-sync_0.0.1_all.deb \
    --output "vyatta-config-sync_0.0.1_all.deb"
    $ sudo dpkg -i vyatta-config-sync_0.0.1_all.deb
    $ sudo curl https://raw.githubusercontent.com/Harliff/vyattta-config-sync/master/scripts/vyatta-config-sync-update.sh \
    --output /opt/vyatta/sbin/vyatta-config-sync-update.sh
    
#### Usage

Start by creating ssh-key:

    mkdir /config/auth/$USER
    ssh-keygen -t rsa -N "" -f /etc/config/$USER/id_rsa
    cat /etc/config/$USER/id_rsa.pub # copy the key
    
Then create an admin user on another router:

    ssh user@<slave-router-ip>
    configure
    edit system login user <username>
    set level admin
    edit authentication public-keys <key name (any text)>
    set type ssh-rsa
    set key <paste you key here>
    commit
    save
    exit
    
Test it!
    
    ssh -i /etc/config/$USER/id_rsa <username>@<slave-router-ip>

Basic synchronization is setup via:

    set system config-sync sync-map slave rule 10 action include
    set system config-sync sync-map slave rule 10 location "nat"

    set system config-sync sync-map slave rule 20 action include
    set system config-sync sync-map slave rule 20 location "firewall"

    # the local user must be able to SSH to the remote host with the specified username
    # with passwordless authentication
    set system config-sync remote-router <remote-host> username <username>
    set system config-sync remote-router <remote-host> sync-map slave

    commit # Will automatically sync config

    # If you want, you can manually run a sync or view the latest status
    run update config-sync <remote-host>
    show config-sync status
