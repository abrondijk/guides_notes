# Enabling email notifcations 

## Details

Attempted to use with my familiebrondijk.nl but fuck that shit cause its shitty.

## Setup

1. Create an app password for gmail [here](https://myaccount.google.com/security)
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

6. Reload the postfix service with `postfix reload`
7. Test whether emails can be send with the `sendmail` command. Exmaple: `sendmail <emailaddr> ENTER test ENTER .`

### **Optional** Remove the sender name from saying root

8. Create a file called `/etc/postfix/smtp_header_checks` with the following contents:

    ```smtp_header_checks
    /^From:.*/ REPLACE From: HOSTNAME-alert <HOSTNAME-alert@something.com>
    ```

9. Add that file to the main.cf by adding:

    ```main.cf
    smtp_header_checks = pcre:/etc/postfix/smtp_header_checks
    ```

10. Finally install the package required for this: `apt install postfix-pcre`

### Links

- <https://forum.proxmox.com/threads/get-postfix-to-send-notifications-email-externally.59940/>
