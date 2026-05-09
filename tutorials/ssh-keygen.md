# how to generate public and private keys and add it to the users trusted public keys

## Step 1: Generate the Key Pair

Open your terminal and run the following command. We use **Ed25519** because it is faster and more secure than the older RSA standard.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/file_name -C "label"
```

* **`-t`**: Specifies the type of key to create.
    > (**ed25519** is the current gold standard)

* **`-f`**: specify the path of the file pair for your key pair

* **`-C`**: Adds a label to the key so you know which one it is late.  
  > (recommended to be the same as `file_name`)

> **Note:** If you are on an older system that doesn't support Ed25519,
  use `ssh-keygen -t rsa -b 4096` instead.
  Then the command would be:

  ```bash
  ssh-keygen -t rsa -b 4096 -f ~/.ssh/file_name -C "label"
  ```

### Step 2: Follow the Prompts

1. **Enter passphrase**: (Optional but recommended) Type a password to protect the key file itself. If you leave it blank, anyone with access to your computer can use the key.

### Step 3: Locate Your Keys

The command creates two files in your `~/.ssh/` directory:

* `file_name`: Your **Private Key**. **Never share this file.**
* `file_name.pub`: Your **Public Key**. This is the one you share with servers.

### Step 4: Get the Public Key Content

To see the text you need to copy-paste into a new line on your server's `authorized_keys` file, run:

```bash
cat ~/.ssh/file_name.pub
```

### Summary of Files

| File Name | Purpose | Analogy |
| --- | --- | --- |
| `file_name` | Private Key (Keep safe!) | The physical key in your pocket. |
| `file_name.pub` | Public Key (Share freely) | The lock on the door. |

> Do note that there are options with CA servers you should use if you
  manage more than 1 person on 3-5 servers, but this procedure doesn't cover that.
