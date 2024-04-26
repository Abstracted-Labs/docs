## Create a new user to run the collator service

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin invarch-collator
```

## Download the latest binary of the InvArch Node

Find the latest release version ending in `-InvArch` at https://github.com/InvArch/InvArch-Node/releases and download it.

```bash
sudo wget -O /usr/local/bin/invarch-collator https://github.com/InvArch/InvArch-Node/releases/download/v1.1.1-InvArch/invarch-collator
sudo chmod +x /usr/local/bin/invarch-collator
sudo chown invarch-collator:invarch-collator /usr/local/bin/invarch-collator
```

## Set up the node

Now that you have a collator node executable at /usr/local/bin/invarch-collator, you should create a data directory for it and download the chainspec file.

Download the chainspec (invarch-raw.json), set up the data directory, and give it the necessary ownership permissions:

```bash
sudo mkdir /var/lib/invarch
sudo wget -O /var/lib/invarch/invarch-raw.json https://raw.githubusercontent.com/InvArch/InvArch-Node/main/res/invarch/invarch-raw.json
sudo chown -R invarch-collator:invarch-collator /var/lib/invarch
```

## Set up a service to run the collator

Create a systemd service file to run your collator (and automatically restart it):

```bash
sudo nano /etc/systemd/system/invarch-collator.service
```

And paste the following in that file (make sure to change `YOUR_COLLATOR_NAME` and customize other parameters if needed):

```yaml
[Unit]
Description=InvArch Collator
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=invarch-collator
Group=invarch-collator
ExecStart=/usr/local/bin/invarch-collator \
  --name YOUR_COLLATOR_NAME \
  --base-path /var/lib/invarch \
  --collator \
  --chain /var/lib/tinkernet/invarch-raw.json \
  --listen-addr "/ip4/0.0.0.0/tcp/30333/ws" \
  --telemetry-url "wss://telemetry.polkadot.io/submit 0" \
  -- \
  --execution wasm
    
Restart=always
RestartSec=120
[Install]
WantedBy=multi-user.target
```

## Running the collator service

After saving the file, run the following commands to start the service:

```bash
sudo systemctl enable invarch-collator
sudo systemctl start invarch-collator.service
```

Your collator node should now be up and running.
If you need to troubleshoot your running service, you can use the `journalctl` command with the `-f` option for tailing:

```bash
journalctl -f -u invarch-collator
```

## Generate your session key

In order to generate keys for your node, run the following command (change the port if you changed it in the service file):

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

Once done, you will have an output similar to:

```json
{"jsonrpc":"2.0","result":"0x9257c7a88f94f858a6f477743b4180f0c9a0630a1cea85c3f47dc6ca78e503767089bebe02b18765232ecd67b35a7fb18fc3027613840f27aca5a5cc300775391cf298af0f0e0342d0d0d873b1ec703009c6816a471c64b5394267c6fc583c31884ac83d9fed55d5379bbe1579601872ccc577ad044dd449848da1f830dd3e45","id":1}
```

## Set your session key

To associate the generated session keys with your Controller account, navigate to the following menu item in the [Polkadot/apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Finvarch-rpc.dwellir.com#/extrinsics) on the Polkadot parachain InvArch: *Developer* > *Extrinsics*.

Fill in the fields:

- *using selected account*: select your Controller account;
- *submit the following extrinsic*: select `session` on the left side and `setKeys` on the right;
- *keys*: enter your session key you just generated;
- *proof*: `0`;

![image](https://github.com/InvArch/docs/assets/30242466/29fbf535-457c-4660-bdd8-00e5f1bb9049)

## Register as a collator candidate

At the moment we are manually onboarding trusted collator nodes, instructions for permissionless registration will be added here in the future.
