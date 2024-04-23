# ssh
Some helpful configs for using ssh

# ssh-copy-id

The `ssh-copy-id` command can be used to add a new public key to a remote `~/.ssh/authorized_keys` file

Suppose you have authorisation to login to a machine `axremote.com` and you've generated a new key that you want to add.

You can say `ssh-copy-id -i new-key axremote.com`

Two notes
- you specify your private key for the `-i` option -- but your public key will be copied.
- `ssh-copy-id` tries to see if there's a key already in the `authorized_keys` file. Sometimes it gets it wrong and you might need to us th `-f` optioin too.

# sauth

You should choose a passphrase when you create a key. However, when you do, if you frequently ssh / scp into a machine it becomes inconvenient _each_ time you do so to specify the pass-phrase. Using an `ssh-agent` is a reasonable compromise between security and ease of use.

Put this file in a convenient place (I'll assume you put it in `~/bin/sauth`. When you log in, before you do an `ssh` run this command `source ~/bin/sauth`. You will be prompted for your passphrase. An agent will  be created on your machine that stores the passphrase on your behalf and when you ssh the agent will provide the passphrase. In this version, the passphrase is stored for 1 day, which is reasonable. When this period expires, you will have re-run `source ~/bin/sauth`. 

# config

If you ssh into multiple machines, it may be tricky to remember which keys you use, or what account names you should use. You can create a file called `~/.ssh/config` and put something like the template into it. In this example, the config file does the following.

1. My default key to use is `~/.ssh/id_ecdsa`. I can specify another key for a specific machine below in the config file or using the `-i` option to ssh.

2. I give some options to reduce the chance of a time-out on my connection and also helping X11 connections (if you don't use X11, remove this)

3. For the machine `ui.core.wits.ac.za` I create a short-hand name `ui`. My account on this machine is `bloggs`. I specify as the hostname the IP address (note that it would make more sense to put the actual host name, but I want to show you  that you can put an IP address)

Now, when I want to log into `ui.core.wits.ac.za`, all I have to is say `ssh ui`. The config file tells ssh to use `~/.ssh/id_ecdsa` as the key (it's the default key) and also tells ssh what the user name is (`bloggs`) and the real host name -- in this case the IP address.

4. At the CHPC, I have an account `jbloggs`; the hostname is `lengau.chpc.ac.za` and here I want to use my `ed25519` key. So, I can now log in by just saying `ssh chpc` and the config file will tell ssh exactly how to do this.

5. Then there's a machine called `proxy.somewhere.com` on which I have an account `jobl`. My default key will be used.

6. Finally, an advanced use which most people will not need. Here I am root on a machine with IP address `192.168.220.139`. I'll give it a short name `inner`. I can't log directly into it (there may be firewall rules), but once I've logged onto `proxy.somewhere.com`, from that machine I _can_ log into the inner machine. So I could just set things up by creating a public/private key on the proxy machine and copying the public key on _proxy_ to the _inner_ `.ssh/authorized_keys` file. However, this is annoying because in order to log into _inner_ I first have to log into _proxy_ and then into _inner_. However, there is an easy solution: using the `ProxyCommand` in the config. Now, when I try to log into inner, ssh first logs on my behalf into _proxy.somewhere.com_ using my rsa key and then logs into _inner_ using the default `ecdsa` key on my _local_ machine. Not only is this more convenient, it is more secure. When I try to log into _inner_ although communication goes via _proxy_ it is all encrypted, including log in details and keys. If for some reason the machine _proxy.somewhere.com_ were compromised, my log in details for _inner_ are not compromised since they do not exist on _proxy_
