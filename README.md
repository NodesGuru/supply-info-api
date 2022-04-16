# Supply Info API

An API for basic info about the any cosmos token supply.

The base route `/` returns all info in JSON:

```json
{
  "apr": 0.5356138011434268,
  "bondedRatio": 0.26138236113619256,
  "circulatingSupply": "1035311800.378731",
  "communityPool": "4863561.388632",
  "denom": "UMEE",
  "totalStaked": "2677292310.820958",
  "totalSupply": "10242819366.934873"
}
```

## Other routes

- `/circulating-supply`: returns circulating supply in plain text
- `/total-supply`: returns total supply in plain text
- `/community-pool`: returns community pool size in plain text
- `/denom`: returns denom in plain text

### How to use?
> ⚠️ This example for [Umee](https://github.com/umee-network) network:

#### Parse vesting accounts:
```bash
cd $HOME
git clone https://github.com/NodesGuru/supply-info-api.git
cd ~/supply-info-api
cp .env.example .env # you can find working .env for umee network in the umee folder
cd umee
wget https://github.com/umee-network/mainnet/raw/main/genesis.json
cat genesis.json | jq -r '.app_state.auth.accounts[] | select(."@type" | contains ("vesting")) | .base_vesting_account.base_account.address' > vesting_accounts_unformatted
sed -i '$ d' vesting_accounts_unformatted
tr '\n' ',' < vesting_accounts_unformatted > vesting_accounts
sed -i '$ s/.$//' vesting_accounts
sed -i '$ d' ~/supply-info-api/.env
echo "VESTING_ACCOUNTS=$(cat vesting_accounts)" >> ~/supply-info-api/.env
```

#### Test with the progress bar:
> :warning: **Do not forget to install NodeJS** and (optional) Yarn
```bash
cd ~/supply-info-api
yarn # or npm install
yarn start # or npm run start
```
Press `Ctrl + C` to exit.

#### Run as `systemd` service:
```bash
echo "[Unit]
Description=Supply Info API
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/supply-info-api
ExecStart=$(which yarn) --cwd $HOME/supply-info-api/ start
Restart=always
RestartSec=1
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target" > $HOME/supply-api-umee.service
sudo mv $HOME/supply-api-umee.service /etc/systemd/system/supply-api-umee.service
sudo systemctl daemon-reload
sudo systemctl enable supply-api-umee
sudo systemctl restart supply-api-umee
journalctl -u supply-api-umee -f -o cat
```

#### Test:
```bash
curl localhost:3000 | jq .
```

### How circulating supply is calculated

1. Get total supply.
2. Get community pool.
3. Subtract community pool from total supply.
4. Iterate through list of vesting amounts for large accounts (like the Dev Fund), and subtract the vesting ammount from total supply.

This yields the circulating supply.

Vesting accounts are provided by an environment variable. See `.env.example` for an example.
