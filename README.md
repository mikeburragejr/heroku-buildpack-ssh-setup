# heroku-buildpack-ssh-setup
Heroku buildpack for setting up the SSH environment (keys, known hosts) before slug compilation (during deployment).

This allows you to use private git repositories (and private servers) without having to checkin credentials.

Commonly, this entails setting up a *deployment* user in your git server (or a [deploy key on github](https://github.com/blog/2024-read-only-deploy-keys)), that has read-only access to some
repositories, and setting heroku config variables to contain the details of that user's SSH private keys.

In this case, the private SSH keys of a read-only user are visible in heroku config for your project, but not
checked into source control.

## Setup

The heroku config variables are **SSH_SETUP_KNOWN_HOSTS** and **SSH_SETUP_KEY_ID_???**

**SSH_SETUP_KNOWN_HOSTS** contains a comma-separated list of base64-encoded versions of the lines from a .ssh/known_hosts file (server name/address, type of server key, hash) for all the private ssh servers in use.

**SSH_SETUP_KEY_???** (ie *SSH_SETUP_KEY_0*, *SSH_SETUP_KEY_1*) contains 3 comma separated attributes - the host name (or IP address or nickname), the SSH user name, and the base64-encoded private key to use.

Then [setup your buildpacks on heroku to call this buildpack first](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app#adding-a-buildpack) to prep the SSH keys prior to bundling.

```sh
heroku buildpacks:add --index 1 https://github.com/mikeburragejr/heroku-buildpack-ssh-setup
```

## Example

Assuming you already have your local SSH client setup with known host info in ~/.ssh/known_hosts and your read-only user SSH private key is at ~/.ssh/read_only_user_id_rsa (having run `ssh-keygen -f ~/.ssh/read_only_user_id_rsa` for example), and the server you want is git.example.com, with the default "git" user in use:

```sh
heroku config:set SSH_SETUP_KNOWN_HOSTS="`grep git.example.com ~/.ssh/known_hosts|base64|tr -d '\n'`"
heroku config:set SSH_SETUP_KEY_0=git.example.com,git,`base64 ~/.ssh/read_only_user_id_rsa|tr -d '\n'`
```

## Notes

- Base64 is used for config variable encoding to keep the config variables to 1 line for ease of use with the heroku CLI commands.
- SSH private keys added must not be password protected or the deployment will fail. Use "openssl rsa -in ~/.ssh/read_only_user_id_rsa -outform PEM" if need be.

