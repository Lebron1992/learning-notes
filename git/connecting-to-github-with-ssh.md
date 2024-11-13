# **connecting to GitHub with SSH**

## Generate a new SSH key
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

## Add your SSH key to the ssh-agent
1. Edit the `~/.ssh/config` file
```bash
Host github.com
AddKeysToAgent yes
UseKeychain yes
IdentityFile ~/.ssh/id_rsa
```

2. Start the ssh-agent in the background
```bash
eval "$(ssh-agent -s)"
> Agent pid 59566
```

3. Add your SSH private key to the ssh-agent
```bash
ssh-add --apple-use-keychain ~/.ssh/id_rsa
```

## Add your SSH key to your GitHub account
1. Copy the SSH key to your clipboard
```bash
pbcopy < ~/.ssh/id_rsa.pub
```
2. Go to **GitHub** > **Settings** > **Access** > **SSH and GPG keys** > **New SSH key**
3. Paste the SSH key to the key field
4. Click **Add SSH key**

## Authorizing an SSH key for use with SAML single sign-on
1. Go to **GitHub** > **Settings** > **Access** > **SSH and GPG keys**
2. Click **Configure SSO**
3. Click **Authorize**
