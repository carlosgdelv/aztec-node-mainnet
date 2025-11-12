# Aztec-node-Mainnet

# 🧰 Aztec Node Setup — Testnet 2.0.2

## Step 1. 🔐 Create folder structure with secure permissions
```bash
mkdir -m 700 -p ~/aztec-sequencer/keys ~/aztec-sequencer/data
```

## Step 2. 📄 Create the .env file
```bash
cd ~/aztec-sequencer
touch .env
```
```bash
nano .env
```

And in `.env`, something like this (adjust if you already have it):

```bash
DATA_DIRECTORY=./data
KEY_STORE_DIRECTORY=./keys
LOG_LEVEL=info
ETHEREUM_HOSTS=[your L1 execution endpoint, or a comma separated list if you have multiple]
L1_CONSENSUS_HOST_URLS=[your L1 consensus endpoint, or a comma separated list if you have multiple]
P2P_IP=[your external IP address]
P2P_PORT=40400
AZTEC_PORT=8080
AZTEC_ADMIN_PORT=8880
```

This command displays the contents of the `.env` file in the terminal.
```bash
cat .env
```

## Step 3. 🏷️ Crete The Aztec Address

Aztec CLI installed:
```bash
bash -i <(curl -s https://install.aztec.network)
```
Run this command in your terminal to temporarily add the directory to the PATH for this current session:
```bash
export PATH="$HOME/.aztec/bin:$PATH"
```
The testnet version installed:
```bash
aztec-up -v latest
```
Set the required environment variables:
```bash
export NODE_URL=https://aztec-testnet-fullnode.zkv.xyz
export SPONSORED_FPC_ADDRESS=0x299f255076aa461e4e94a843f0275303470a6b8ebe7cb44a471c66711151e529
```
Unlike sandbox, testnet has no pre-deployed accounts. You need to create your own:
```bash
aztec-wallet create-account \
    --register-only \
    --node-url $NODE_URL \
    --alias my-wallet
```
# Step 4. 🔐 Keystore Encryption (Attester)

1️⃣ Create a file with your private key (without 0x)
```bash
printf "aabb...887799" > /tmp/privatekey.txt
chmod 600 /tmp/privatekey.txt
```
2️⃣ Crear un archivo con la contraseña de cifrado
Create a file with the encryption password
Disable shell history (prevents the commands from being saved in `~/.bash_history`)
```bash
set +o history
```
3️⃣ Generate and display a 10-word passphrase ONE TIME ONLY (it is not saved to a variable or file):
```bash
grep -E '^[a-z]{5,}$' /usr/share/dict/words | shuf -n 10 | paste -sd ' ' -
```
## 4️⃣ WRITE THE PASSPHRASE ON PAPER
Confirm before continuing:

```bash
read -s -p "Apunta la passphrase en papel y pulsa ENTER para continuar..." ; echo
```

5️⃣ Clear the screen and scrollback (works in most modern terminals)
```bash
printf '\033c'
printf '\e[3J' 
clear
```
6️⃣ Restore shell history:
```bash
set -o history
```
7️⃣ Save the passphrase to a secure file
```bash
read -s -p "Introduce ahora la passphrase que escribiste en papel: " PASSWORD
echo
printf "%s" "$PASSWORD" > ~/aztec-sequencer/password.txt
chmod 600 ~/aztec-sequencer/password.txt
unset PASSWORD
```
8️⃣ Verify there is no trailing newline
```bash
hexdump -C ~/aztec-sequencer/password.txt | tail -n1
```

9️⃣ Check file permissions (DOES NOT display the passphrase)
```bash
ls -l ~/aztec/password.txt
wc -c ~/aztec/password.txt
```

🔟  Import the private key as an encrypted keystore
```bash
geth account import --keystore ~/aztec-sequencer/keys --password ~/aztec-sequencer/password.txt /tmp/privatekey.txt
```

⓫  Remove the temporary private key
```bash
shred -u /tmp/privatekey.txt
```
🔍 How to know exactly which file was created (and its path)

Directly from geth with the account list
```bash
geth account list --keystore ~/aztec-sequencer/keys
```
You will see something like:
```bash
Account #0: {0xabcdef1234567890} /home/usuario/aztec-sequencer/keys/UTC--2025-10-22T17-41-12.123Z--0xabcdef1234567890.json
```
That second value is exacly the `path` you must use in your `validators.json`.

## 📄 Step 5. — Configure your validators.json

Edit your JSON to point to that file and to the password. If only the attester is encrypted, and the other roles (`coinbase`,`publisher` ) are plaintext, it would look like this: Edita tu JSON para que apunte a ese archivo y a la contraseña.

```bash
{
  "schemaVersion": 1,
  "validators": [
    {
      "attester": {
        "path": "/home/usuario/aztec-sequencer/keys/UTC--2025-10-16T22-40-30.000Z--0xabcdef1234567890.json",
        "password_file": "/home/usuario/aztec-sequencer/password.txt"
      },
      "publisher": "0x1234567890abcdef1234567890abcdef12345678",
      "coinbase": "0x9876543210abcdef9876543210abcdef98765432",
      "feeRecipient": "0xabcdef1234567890abcdef1234567890abcdef12"
    }
  ]
}

```

🔐 NOTE:
Use "password_file" (not "password") if the software allows it — this way you don't leave the password written in the JSON in plain text. If it only accepts "password", you can leave it in clear text, but it is less secure.


## 🛡️ Step 6. Security and verification

Apply strict permissions:
```bash
chmod 700 ~/aztec-sequencer/keys
chmod 600 ~/aztec-sequencer/keys/*
chmod 600 ~/aztec-sequencer/password.txt
```

## Step 7. 🧪 Verifications

✅ Verify that the file exists

```bash
ls ~/aztec-sequencer/keys/UTC--*
```

✅ Verify that the keystore was imported:
```bash
geth account list --keystore ~/aztec-sequencer/keys
```

✅ Verify that the validators.json file is well-formed:
```bash
jq . ~/aztec-sequencer/validators.json
```

✅ Verify that you can read the address from the JSON
```bash
jq -r .address ~/aztec-sequencer/keys/UTC--*.json
```

✅ Verify the KDF encryption parameters:
```bash
jq .crypto.kdfparams ~/aztec-sequencer/keys/UTC--*.json
```

✅  Check the permissions of the password file
```bash
ls -l ~/aztec-sequencer/password.txt
```

✅ Check whether the password.txt file contains a newline (it should not)
```bash
hexdump -C ~/aztec-sequencer/password.txt | tail -n1
```
If it ends with 0a => newline, rewrite with printf

Recommended values:
```bash
"n" ≥ 262144 (cuanto más alto, más lento el brute force)
"r" ≥ 8
"p" ≥ 1
```

## Step 8. 🔧  Directory and user permissions

✅ Verify:
```bash
ls -l ~/aztec-sequencer
ls -l ~/aztec-sequencer/keys
id $USER

```
✅ That is correct if carlos has UID 1000. 

To confirm:
```bash
id carlos
```
🛠️  If for some reason the permissions are not correct, fix it:
```bash
sudo chown -R 1000:1000 ~/aztec-sequencer
```

## 🐳 9. Configure docker-compose.yml
Edit:
```bash
nano docker-compose.yml
```
Replace the following code into your `docker-compose.yml` file:
```yaml
services:
  aztec-sequencer:
    image: "aztecprotocol/aztec:2.0.2"
    container_name: "aztec-sequencer"
    network_mode: host
    user: "1000:1000"
    ports:
      - ${AZTEC_PORT}:${AZTEC_PORT}
      - ${AZTEC_ADMIN_PORT}:${AZTEC_ADMIN_PORT}
      - ${P2P_PORT}:${P2P_PORT}
      - ${P2P_PORT}:${P2P_PORT}/udp
    volumes:
      - ${DATA_DIRECTORY}:/var/lib/data
      - ${KEY_STORE_DIRECTORY}:/var/lib/keystore
    environment:
      KEY_STORE_DIRECTORY: /var/lib/keystore
      DATA_DIRECTORY: /var/lib/data
      LOG_LEVEL: ${LOG_LEVEL}
      ETHEREUM_HOSTS: ${ETHEREUM_HOSTS}
      L1_CONSENSUS_HOST_URLS: ${L1_CONSENSUS_HOST_URLS}
      P2P_IP: ${P2P_IP}
      P2P_PORT: ${P2P_PORT}
      AZTEC_PORT: ${AZTEC_PORT}
      AZTEC_ADMIN_PORT: ${AZTEC_ADMIN_PORT}
    entrypoint: >-
      node
      --no-warnings
      /usr/src/yarn-project/aztec/dest/bin/index.js
      start
      --node
      --archiver
      --sequencer
      --network testnet
    restart: always
```
## 📂 10. Secure permissions of the mounted volumes

✅ Verify that the local directories (./data and ./keys) have permissions for UID 1000:
```bash
sudo chown -R 1000:1000 ~/aztec-sequencer/data
sudo chown -R 1000:1000 ~/aztec-sequencer/keys
```

## 🚀 11. Start & update the node

✅ Validate before bringing it up:
```bash
geth account list --keystore ~/aztec-sequencer/keys
```
▶️ Start the services:
```bash
docker compose up -d
```
🧾 View logs in real time:

```bash
docker compose logs -f
```
⛔ Stop the node:
```bash
docker compose down
```

🔄 Update Node:

Run `aztec-up alpha-testnet` to fetch the latest configuration files for the Alpha Testnet environment.
```bash
aztec-up alpha-testnet
```
Run `docker compose pull` to download the most recent Docker images defined in your Compose setup.
```bash
docker compose pull
```
Delete old data:
This command forcefully deletes the entire Aztec alpha-testnet data directory and all its contents from your home folder to reset the node.
```bash
rm -rf ~/.aztec/alpha-testnet/data/
```
Re-run Node
