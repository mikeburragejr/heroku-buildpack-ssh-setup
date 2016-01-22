# heroku-buildpack-ssh-setup
Heroku buildpack for setting up the SSH environment (keys, known hosts) before slug compilation (during deployment).

This allows you to use private git repositories (and private servers) without having to checkin credentials.

Commonly, this entails setting up a _deployment_ user in your git server, that has read-only access to some
repositories, and setting heroku config variables to contain the details of that user's SSH private keys.

In this case, the private SSH keys of a read-only user are visible in heroku config for your project, but not
checked into source.

## Setup

The heroku config variables are *SSH_SETUP_KNOWN_HOSTS* and *SSH_SETUP_KEY_ID_???*

*SSH_SETUP_KNOWN_HOSTS* contains a comma-separated list of base64-encoded versions of the lines from a .ssh/known_hosts file (server name/address, type of server key, hash) for all the private ssh servers in use.

*SSH_SETUP_KEY_???* (ie SSH_SETUP_KEY_0, SSH_SETUP_KEY_1) contains 3 comma separated attributes - the host name (or IP address), the SSH user name to be user, and the base64-encoded private key to use.

Then set your buildpack on heroku to this buildpack first (to prep the SSH keys prior to bundling).

```sh
heroku buildpacks:add --index 1 https://github.com/mikeburragejr/heroku-buildpack-ssh-setup
```

## Example

Assuming you already have your local SSH client setup with known host info in ~/.ssh/known_hosts and your read-only user SSH private key is at ~/.ssh/read_only_user_id_rsa (having run `ssh-keygen -f ~/.ssh/read_only_user_id_rsa` for example), and the server you want is git.example.com, with the default "git" user in use:

```sh
heroku config:set SSH_SETUP_KNOWN_HOSTS="`grep git.example.com ~/.ssh/known_hosts|base64|tr -d '\n'`"
heroku config:set SSH_SETUP_KEY_0=git.example.com,git,`base64 ~/.ssh/read_only_user_id_rsa|tr -d '\n'`
```
