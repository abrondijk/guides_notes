# Enabling email notifcations 

## Details

Attempted to use with my familiebrondijk.nl but fuck that shit cause its shitty.

## Setup

1. Create an app password for gmail [here]()
2. Create a password file in `/etc/postfix/` with the following contents:

    ```password
    [smtp.gmail.com]:587    <account>@gmail.com:<app_password>
    ```

3. Create a postfix lookup table from the password file: `postmap /etc/postfix/<password_file>`
4. Install requirements: `apt install libsasl2-modules`
5. Edit the `/etc/postfix/main.cf` file to containt the following lines:

    ```main.cf
    relayhost = [smtp.gmail.com]:587
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_security_options = noanonymous
    smtp_sasl_password_maps = hash:/etc/postfix/<password_file>
    smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
    inet_protocols = ipv4
    mynetworks = 127.0.0.0/8 <other_networks_subnets>
    mailbox_size_limit = 0
    recipient_delimiter = +
    inet_interfaces = all
    ```





### Links

- https://forum.proxmox.com/threads/get-postfix-to-send-notifications-email-externally.59940/