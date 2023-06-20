#             Skill Factory Linux Admin Final project

   These ansible roles installs all services need on additional servers:
u-agent: nginx, Apache, PHP, Zabbix-agent, bind, mail service and filebeat.
db-server: PostgreSQL-12, Zabbix-agent, ELK stack

All roles may use files from dir /et/ansible/common. It's contents is:
../common
    apache2
        mymail.conf       # create your own site
        roundcube.conf    # alias for dovecot
    bind
        99-disable-network-config.cfg
        named.conf.local      # configure your DNS
        named.conf.options    # DNS options
        sf-alek.db            # describe your zone
        sf-alek-reverse.db    # your reverse zone
    dovecot
        conf.d                # this folder includes ready configs
            10-auth.conf
            10-mail.conf
            10-master.conf
            auth-passwdfile.conf.ext
        dovecot-users         # define users
    ELK_deb
        <filebeat_deb>        # provide desired deb pkg, change the name in vars.yml
        filebeat.yml          # ready config
        kibana.yml            # ready config
    ELK_rpm
        <elasticsearch.rpm>   # provide desired rpm, change vars.yml
        elasticsearch.yml     # ready config
        <filebeat.rpm>        # provide prm, change vars.yml
        <GPG-KEY-elasticsearch.skr>   # receive GPG key and place here
        <kibana.rpm>          # desired rpm for kibana, change vars.yml
        kibana.yml            # ready to use
        <logstash.rpm>        # load rpm for logstash, change vars.yml
    nginx                     # ready cinfig for kibana
        kibana.conf
        .htpasswd
    postfix                   # this folder contains all configs for postfix
        main.cf
        master.cf
        virtual_mailbox_domains
    roundcube
        roundcube.conf        # roundcube config

#                     Install order
#     u-agent
The role ubuntu_net_inst.yml includes:
            openvpn_inst.yml
            bind_inst.yml

After they were run, connect to u-agent server and configure netplan:
   /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg must include: network: {config: disabled}
   edit /etc/netplan/50-cloud-init.yaml to add:
         . . . . . . . . . .
         set-name: eth0
         nameservers:
             addresses: [10.0.0.2]
             search: [sf-alek.ru]
         . . . . . . . . . .
   perform: sudo netplan apply
            sudo systemctl restart bind9.service

Role ubuntu_full_inst.yml includes:
            web_inst.yml
            postfix_inst.yml
            mariadb_inst.yml    ; only installs DB, set root password manually
            dovecot_inst.yml
            zabbix_agent_inst.yml
            filebeat_inst.yml

