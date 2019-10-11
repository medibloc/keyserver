# Keyserver

This is a basic `keyserver` for cosmos-sdk applications. It contains the following routes:

```
GET     /version
GET     /keys
POST    /keys
GET     /keys/{name}?bech=acc
PUT     /keys/{name}
DELETE  /keys/{name}
POST    /tx/sign
POST    /tx/bank/send
POST    /tx/broadcast
```

First, build and start the server:

```bash
> make install
> keyserver config
> keyserver serve
```

Then you can use the included CLI to create keys, use the mnemonics to create them in `panaceacli` as well:

```bash
# Create a new key with generated mnemonic
> keyserver keys post ggomma foobarbaz | jq

# Create another key
> keyserver keys post min foobarbaz | jq

# Save the mnemonic from the above command and add it to panaceacli
> panaceacli keys add ggomma --recover

# Next create a single node testnet
> panacead init testing --chain-id testing
> panaceacli config chain-id testing
> panacead add-genesis-account ggomma 10000000000000000umed
> panacead add-genesis-account $(keyserver keys show min | jq -r .address) 100000000000000umed
> panacead gentx --name ggomma
> panacead collect-gentxs
> panacead start
```

In another window, generate the transaction to sign, sign it and broadcast:
```bash
> mkdir -p test_data
> keyserver tx bank send $(keyserver keys show ggomma | jq -r .address) $(keyserver keys show min | jq -r .address) 10000umed testing "memo" 10umed > test_data/unsigned.json
> keyserver tx sign ggomma foobarbaz testing 0 1 test_data/unsigned.json > test_data/signed.json
> keyserver tx broadcast test_data/signed.json
{"height":"0","txhash":"BA7D9E1029E2896A909D4FC33B1BE2AEEF7EC467CF30BDD564B2AF94EFC579CB"}
> panaceacli q tx BA7D9E1029E2896A909D4FC33B1BE2AEEF7EC467CF30BDD564B2AF94EFC579CB
```

When offline, transaction gas can be set manually:
```
> keyserver tx bank send $(keyserver keys show ggomma | jq -r .address) $(keyserver keys show min | jq -r .address) 10000umed testing "memo" 10umed 1 200000 > test_data/unsigned.json
```
