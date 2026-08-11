# SSH Remote Server Setup

This project documents the process of setting up secure, key-based SSH access to a remote Linux server. It covers generating multiple SSH key pairs, adding them to the server, connecting with each key manually, and configuring `~/.ssh/config` to simplify future logins with short aliases.

> **Note:** No private keys are included in this repository. Only this README and (optionally) the public keys are safe to commit.

---

## Project Objectives

- Set up a remote Ubuntu server accessible via SSH
- Generate two separate SSH key pairs
- Add both public keys to the server's authorized keys
- Connect to the server using each key individually with `ssh -i`
- Configure `~/.ssh/config` to connect using short aliases (`ssh ubuntu1`, `ssh ubuntu2`)

---

## Environment

| Component        | Details                          |
|-------------------|-----------------------------------|
| Server OS         | Ubuntu Server (remote/VM)         |
| Client OS         | *(e.g. Windows / macOS / Linux)*  |
| Server IP         | `<SERVER_IP>`                     |
| SSH Port          | `22` (default)                    |
| Server Username   | `<USERNAME>`                      |

Replace the placeholders above with your actual server details before publishing, or remove the table if you'd rather not disclose them.

---

## Step 1: Set Up the Remote Ubuntu Server

1. Installed Ubuntu Server on the remote machine (or VM).
2. Enabled the OpenSSH server during installation (or installed it afterward with `sudo apt install openssh-server`).
3. Verified the SSH service was running:
   ```bash
   sudo systemctl status ssh
   ```
4. Noted the server's IP address using:
   ```bash
   ip a
   ```
5. Confirmed the server was reachable from the client machine:
   ```bash
   ping <SERVER_IP>
   ```

---

## Step 2: Generate Two SSH Key Pairs

SSH key pairs consist of a **private key** (kept secret, stays on the client) and a **public key** (shared with the server). Two separate key pairs were generated to simulate two different identities/use cases.

### Key Pair 1
```bash
ssh-keygen -t ed25519 -f ~/.ssh/ubuntu1_key -C "ubuntu1-key"
```

### Key Pair 2
```bash
ssh-keygen -t ed25519 -f ~/.ssh/ubuntu2_key -C "ubuntu2-key"
```

**Explanation of flags:**
- `-t ed25519` — uses the Ed25519 algorithm, which is modern, fast, and more secure than older RSA keys at smaller key sizes.
- `-f` — specifies the output file path/name for the key pair.
- `-C` — adds a comment (usually an identifier) to help distinguish the key later.

Each command produces two files:
- `ubuntu1_key` / `ubuntu2_key` — the **private** key (never shared, never pushed to GitHub)
- `ubuntu1_key.pub` / `ubuntu2_key.pub` — the **public** key (safe to share)

A passphrase was set on each key during generation for an extra layer of security.

---

## Step 3: Add Both Public Keys to the Server

There are two common ways to copy a public key to a server. Both were used here to demonstrate each method.

### Method A — Automatic (`ssh-copy-id`)
This is the fastest method when password authentication is still enabled on the server:
```bash
ssh-copy-id -i ~/.ssh/ubuntu1_key.pub <USERNAME>@<SERVER_IP>
ssh-copy-id -i ~/.ssh/ubuntu2_key.pub <USERNAME>@<SERVER_IP>
```
`ssh-copy-id` automatically appends the public key to the server's `~/.ssh/authorized_keys` file and sets the correct file permissions.

### Method B — Manual
Useful when `ssh-copy-id` isn't available, or to understand what's happening under the hood:

1. Display the public key content locally:
   ```bash
   cat ~/.ssh/ubuntu1_key.pub
   ```
2. Log in to the server (using a password, or console access) and create the `.ssh` directory if it doesn't exist:
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```
3. Append the copied public key to the `authorized_keys` file:
   ```bash
   echo "<paste-public-key-here>" >> ~/.ssh/authorized_keys
   ```
4. Set correct permissions (SSH is strict about this — overly permissive files will cause SSH to reject key-based logins):
   ```bash
   chmod 600 ~/.ssh/authorized_keys
   ```
5. Repeated the same steps for the second public key (`ubuntu2_key.pub`).

---

## Step 4: Connect Using Each Key with `ssh -i`

Verified that both keys worked independently before setting up any shortcuts:

```bash
ssh -i ~/.ssh/ubuntu1_key <USERNAME>@<SERVER_IP>
```

```bash
ssh -i ~/.ssh/ubuntu2_key <USERNAME>@<SERVER_IP>
```

The `-i` flag tells SSH exactly which private key to use for authentication instead of relying on the default (`~/.ssh/id_rsa` or `~/.ssh/id_ed25519`). Both connections succeeded without requiring a password, confirming the keys were correctly installed on the server.

---

## Step 5: Configure `~/.ssh/config` for Short Aliases

To avoid typing the full `ssh -i <path> user@ip` command every time, an SSH config file was created (or edited) at `~/.ssh/config`:

```ssh-config
Host ubuntu1
    HostName <SERVER_IP>
    User <USERNAME>
    IdentityFile ~/.ssh/ubuntu1_key
    Port 22

Host ubuntu2
    HostName <SERVER_IP>
    User <USERNAME>
    IdentityFile ~/.ssh/ubuntu2_key
    Port 22
```

**Explanation of each field:**
- `Host` — the alias you'll type (e.g. `ssh ubuntu1`)
- `HostName` — the actual IP address or domain of the server
- `User` — the username to log in as
- `IdentityFile` — which private key to use for this host
- `Port` — the SSH port (default is 22, only needed if different)

Set correct permissions on the config file itself:
```bash
chmod 600 ~/.ssh/config
```

---

## Step 6: Connect Using the Aliases

With the config file in place, connecting became as simple as:

```bash
ssh ubuntu1
```

```bash
ssh ubuntu2
```

Both aliases connected successfully, automatically using the correct private key, username, host, and port — no extra flags required.

---

## Security Notes

- **Private keys were never committed to this repository.** Only this README (and optionally the `.pub` files, which are safe to share) are included.
- A `.gitignore` was used locally to make sure key files are excluded by accident:
  ```gitignore
  *.pem
  ubuntu1_key
  ubuntu2_key
  id_rsa
  id_ed25519
  ```
- Passphrases were used on both private keys for defense-in-depth.
- File permissions on `~/.ssh` (700), `authorized_keys` (600), and `config` (600) were verified, since SSH silently ignores files with overly permissive access.

---

## Summary

This project demonstrated end-to-end SSH key-based authentication: generating multiple key pairs, deploying public keys to a remote Ubuntu server both manually and automatically, verifying access with explicit key flags, and finally streamlining the workflow with SSH config aliases for fast, repeatable access.
