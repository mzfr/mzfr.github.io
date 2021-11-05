---
layout: post
title: Introduction to Cardano components
date: "2021-11-05"
ShowToc: true
tags:
    - cardano
    - NFT
---

A few months ago one of my friends asked me to help him mint some NFT on the Cardano blockchain. At that point, I had no idea how transactions work on Cardano and how to actually mint any NFT on it. So I started reading all the blog posts I could find and started banging my head on the Cardano documentation.

Now I'm writing this series (hopefully) of blog posts because I had to learn things from different places and couldn't find everything in the Cardano documentation because it's not very frequently updated. I believe the main reason for that is because Cardano is under very active development so most of the time devs focus on making the application instead of updating the documentation.

These blog posts will be my way of sharing the knowledge I've gained over these few months and probably help the newcomers planning to start in these.

# What is Cardano?

According to [cardano.org](https://cardano.org/)

> Cardano is a blockchain platform for changemakers, innovators, and visionaries, with the tools and technologies required to create the possibility for the many, as well as the few, and bring about positive global change

In simple terms, it is one of the many blockchain platforms which has recently come into the limelight (I think). And is believed to be a competitor of ETH i.e do everything ETH can do but with less gas fee.

Without getting into the detail of why Cardano was made, how it's different, etc. I'm just going to jump into how one can start their own [cardano-node](https://github.com/input-output-hk/cardano-node) and then interact with using `cardano-cli` to submit transactions, mint/burn/evolve tokens.

# Running own cardano-node

In this, we are going to run our own `testnet`. But what is `testnet`?

To quote the `cardano.org`

> The Cardano testnet offers full Cardano functionality, including stake pool operation, transaction metadata, native tokens, and more.

Basically, on testnet we can do exactly what can be done in the real-world but without using any real money(ADA).

Before we start I'd also like to point out that the almost everything will be similar in case you decide to run the `mainnet`. And I'll point out the things that could be different in the case of mainnet.

## Starting Testnet

1. Go to [releases page](https://github.com/input-output-hk/cardano-node/releases) of the [cardano-node](https://github.com/input-output-hk/cardano-node/) repository. And in the `download` section you'll see the `Hydra binaries` and the `Configurations files`.
    - Remember to download configuration files for `testnet` and **NOT** mainnet.
    - Only download the `config`, `byronGenesis`, `ShelleyGenesis`, `alozonGenesis` and `topology`.
    - Store all the config files in a folder named `configs`
    - We **don't** need `db-sync config` and `rest config` for running the node.

2. To start the node run the following command:

```shell
cardano-node run \
    --topology ./configs/testnet-topology.json \
    --database-path ./db --port 3001 \
    --config ./configs/testnet-config.json \
    --socket-path testnet.socket
```

- `--topology` - path to the `testnet-topology.json`. This basically pushes the node alive message, showing that a new node has come into the network.
- `--database-path` - This can be any path. Basically here all synced node information will be stored.
- `--config` - path to the testnet config.
- `--socket-path` - the path where you want to create the `testnet.socket`
    - If you are on windows then this will be changed to, `--socket-path \\.\\pipe\\cardano-node`
- `--port` - This can be any port you want.

3. Now run the following command:

```bash
 export CARDANO_NODE_SOCKET_PATH="/path/to/your/testnet.socket"
```

If you are on Windows then run:

```bash
$env:CARDANO_NODE_SOCKET_PATH='\\.\\pipe\\cardano-node'
```

Remember the above commands will only set variables in your current shell and not globally. If you'd like to set variables globally then:

- For Linux - add this (`export CARDANO_NODE_SOCKET_PATH="/path/to/your/testnet.socket"`) in your .bashrc/.zshrc
- For Windows - read [this](https://www.architectryan.com/2018/08/31/how-to-change-environment-variables-on-windows-10/) to see how to add Environment variables.

4. Now you'll have to wait for a few minutes and then run the following command:

```bash
cardano-cli query tip --testnet-magic 1097911063
```

The `testnet-magic` number can be found in the `testnet-shelley-genesis.json` file.

Now if everything was done correctly, then with the above command you must see something like:

```json
{
    "epoch": 166,
    "hash": "c4060a65975397e84a5f59f73e09c5f857339f661a7db7389e6f5b76cb75b89a",
    "slot": 41764705,
    "block": 3049124,
    "era": "Alonzo",
    "syncProgress": "100.00"
}
```

The values might vary but if you get similar-looking output then your node is up and running and is being synced.

__NOTE__: The testnet network takes about 4-5 hours to sync 100% and the mainnet network takes about 24 hours to sync.

When you run the command given in step 4 and get an error saying something like:

```
cardano-cli: Network.Socket.connect: <socket: 11>: does not exist (Connection refused)%
```

Then please wait for some time before re-trying the same command, sometimes it takes a bit longer to generate the `socket` file.

# Create simple transactions

In this blog post, we are only going to focus on transferring tADA(testing ADA) from one wallet to another wallet. In the future, I'll explain on how to mint NFTs on the Cardano chain and how to transfer them.

But before we get into making transactions we need to generate a payment address.

## Generating payment address

1. Make a new directory called `payment_addr1`

```bash
mkdir -p $HOME/payment_addr1/
```

2. Now generate a payment key-pair

```bash
cardano-cli address key-gen \
    --verification-key-file $HOME/payment_addr1/payment.vkey \
    --signing-key-file $HOME/payment_addr1/payment.skey
```

3. Now using the above key-pair we can generate a wallet address

```bash
cardano-cli address build \
    --payment-verification-key-file $HOME/payment_addr1/payment.vkey \
    --out-file $HOME/payment_addr1/payment.addr --testnet-magic 1097911063
```

I'd suggest generating another address (under `payment_addr2`) in a similar manner. This way you can perform all the transactions and see things happening.

## Getting tADA in the wallet

Now that we have the wallet, you can run the following command to see if there are any tADA in your wallet (there shouldn't be any)

```bash
cardano-cli query utxo --address <your_payment_addr1_address_here> --testnet-magic 1097911063
```

You should see something like:

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
```

This shows that you don't have any tADA in your wallet. Now to get some tADA to go to, [testnet faucet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/) and enter your `wallet address`. You don't need an API key(it's not compulsory to put it there). You can request any amount of tADA you want, for this tutorial we are going to request `20 tADA`.

Once you've requested the funds, wait for a few minutes and run the following command:

```bash
cardano-cli query utxo --address <your_payment_addr1_address_here> --testnet-magic 1097911063
```

you should see an output that looks like the following:

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
4606b0a96e0b69569be47293fdfcebda19da3fd2b7c0066d695f13020663c524     1        20000000 lovelace + TxOutDatumHashNone
```

## What is a UTxO

We requested 10 tADA but in the above output, we can see a `TxHash`, `TxIx`, and `Amount`. This as a whole is known as UTxO(Unspent Transaction Output). The concept of UTxO is taken from Bitcoin. 

A UTxO represents a Transaction that isn't spent, hence the name `unspent`. This means that chain knows that in the whole network there is a UTxO with a certain unique identification (combination of `TxHash` & `TxIx`) which stores some unspent amount. Now, this UTxO(s) can be used to make transactions i.e to transfer the amount. The UTxO model in Cardano/BTC varies from what a chain like ETH uses. In ETH it's what we are used to, A has 10 in their wallet and would like to send 5 to B, so when a transaction on ETH will be submitted it will reduce 5 from A and increase 5 on the B side. But a UTxO model **doesn't** work like that. In a UTxO model, a whole UTxO is consumed when a transaction is submitted. 

**Ex**: Say **A** has a UTxO with amount `10` tADA and would like to send 5 tADA to **B** and keep the remaining 5 tADA. Now in Cardano, a simple increment on B's amount and decrement on A's amount, won't happen. Instead while making a transaction a UTxO(s) will be given as an input, which stores the 10 tADA. Once that transaction is submitted, that UTxO will be consumed completely and a new UTxO with the same TxHash but different TxIx will be generated. One of the UTxO will be present in A's wallet showing 5 tADA remaining and another UTxO(same TxHash but different TxIx) will be shown in B's wallet.

__NOTE__: In the above example I've not considered the gas fee.

Understanding UTxO can be a difficult task, if you like to further read about it then I'd suggest Cardano's documentation on [UTxO models](https://developers.cardano.org/docs/stake-pool-course/handbook/utxo-model/)

And in the UTxO we see the amount written in Lovelace. 
    
    1 ADA/tADA == 1000000 Lovelace

So when we transferred 20 tADA we got `20000000` Lovelace.

## Making a transaction

Now that we understand what a UTxO is and also have a proper setup of tools. We can start making transactions. There are 3 steps to submit a transaction:

1. Build a transaction: We provide the UTxO input and the output address along with the output and some other arguments.
2. Sign a transaction: We then sign the transaction, to show that we actually own the UTxO which will be consumed.
3. Submit the transaction

### Building a transaction

In order to build a transaction we are going to use the following command:

```bash
cardano-cli transaction build --alonzo-era \
    --tesnet-magic 1097911063 \
    --tx-in $TX_IN \
    --tx-out $ADDR+$AMOUNT \
    --change-address $OWN_ADDR \
    --out-file tx.raw
```

- `--alonzo-era`: This is the latest cardano era. You can also use `--byron-era` or `--shelley-era` but those might give errors.

- `--testnet-magic`: If we were on `mainnet` then this would have changed to `--mainnet`. And with mainnet we don't need to provide any magic number.

- `--tx-in`: This is where we input the UTxO which has to be consumed. 
    + In a single command, there can be as many `--tx-in` argument as you want. 
        * **Ex:** Say you want to send 5 tADA but you have 3 tADA in X UTxO and 2 tADA in Y UTxO, so you can give 2 `--tx-in` so both of them can be consumed.
    + Another thing to remember is that the `$TX_IN` format would be **TxHash#TxIx** i.e if the `TxHash` was `4606b0a96e0b69569be47293fdfcebda19da3fd2b7c0066d695f13020663c524` and `TxIx` was `1` then we would write `--tx-in 4606b0a96e0b69569be47293fdfcebda19da3fd2b7c0066d695f13020663c524#1`, notice the `#1` in the end.

- `--tx-out`: Here we will put the address of the person we want to send the amount along with the amount, separated by `+`. 
    + Just the way we can have multiple `--tx-in` in the same way we can have multiple `--tx-out`
    + The format for `--tx-out` will be `<ADDR_TO_SEND_tADA>+<AMOUNT_TO_BE_SENT>`
    + Amount of ada has to be in Lovelace.

- `--change-address`: This will be the address to which the changes i.e the remaining amount, will be sent. 
    + Say you have 5 tADA in the UTxO and you'd like to transfer 3 tADA and would like to have 2 tADA back. So in the `--change-address` you can put your own address, so once the transaction is submitted 3 tADA will be transferred,`~0.2` will be paid as the gas fee and the remaining 1.8 tADA(changes) will be sent back to you.

- `--out-file`: The output file which will hold all the transaction information.

Now I'm going to use 20 tADA that I got from testnet faucet to build a transaction:

```bash
cardano-cli transaction build --alonzo-era --tesnet-magic 1097911063 \
     --tx-in 4606b0a96e0b69569be47293fdfcebda19da3fd2b7c0066d695f13020663c524#1 \
     --tx-out $payment_addr2+10000000 --change-address $payment_addr1 --out-file tx.raw
```

In the above transaction I'm sending `10 tADA` from `payment_addr1` -> `payment_addr2`. When I ran the above command I got a file called `tx.raw`

### Signing a transaction

Here we are going to sign the transaction which we generated in the [building a transaction](#building-a-transaction).

```bash
cardano-cli transaction sign \
    --signing-key-file $HOME/payment_addr1/payment.skey \
    --testnet-magic 1097911063 --tx-body-file tx.raw --out-file tx.signed
```

- `--signing-key-file`: This is the argument in which we provide our wallet's `skey`
- `--tx-body-file`: This takes the path to the file which we get from `building a transaction`.

**Ex:** I'm going to sign the transaction which I built using the `payment_addr1` skey. We have to use `payment_addr1` keys because we are sending it from that wallet.

```bash
cardano-cli transaction sign \
    --signing-key-file $HOME/payment_addr1/payment.skey \
    --testnet-magic 1097911063 --tx-body-file tx.raw --out-file tx.signed
```

### Submit a transaction

This is probably the easiest step. Just run the following command:

```bash
cardano-cli transaction submit --tx-file tx.signed --testnet-magic 1097911063
```

- `--tx-file`: Path of the file which we got after we `signed a transaction.`

Once we run that we will get the acknowledgment that the transaction is submitted successfully.

***

So this was all you need to do/know to get started with cardano-node and cardano-cli. In the next post, I'll try to explain how to mint an NFT and how to transfer them between wallets.

Thanks for reading, feedback is always appreciated :)

You can follow me on [@0xmzfr](https://twitter.com/0xmzfr)