## Description
This service main purpose is to be a "balancing" tool for lnsov bridge.  
The lnsov bridge is an exchange that offers rbtc / lightning swaps. It should maintain ~50% of funds in lightning wallet and 50% funds in rsk wallet.  
It uses lightning loop service to do either:
 - loop in 
   - when lightning balance falls below X%
     1. this service sends funds from rsk wallet to RSK federation address
     2. after 100 confirmations RSK federation address sends funds to "rsk-paired" bitcoin wallet 
     3. this service is monitoring "rsk-paried" bitcoin address for new transactions
     4. after new transaction is found it sends it to lightning wallet
     5. after funds arrive to lightning wallet it initiates a loop in
 - loop out
   - when RBTC balance falls below X%
     1. this service initiates a loop out 
     2. loop out service sends funds to our lightning wallet
        - **it is possible to loop out to specific bitcoin address, so by modfying step1, we might skip straight to step4 and save 1 onchain transaction**
     3. this service monitors loop service for loop SWAP status, 
     4. when swap status=COMPLETE it sends onchain transaction to "rsk-paired" bitcoin wallet
     5. this service sends coins from "rsk-paired" wallet to RSK-Federation address
     6. after 100 confirmations funds are available in RSK wallet

This service requires access to the following information:  
 - RSK wallet private key
 - RSK-paired bitcoin private key 
   - https://github.com/rsksmart/utils
   - https://developers.rsk.co/rsk/rbtc/conversion/networks/mainnet/
 - Lightning loop client macaroon

```properties
# RSK NODE URL FOR MONITORING RSK CHAIN AND SENDING TRANSACTIONS
rsk.service.url=${RSK_SERVICE_URL}
# RSK WALLET PUBLIC ADDRESS FOR MONITORING BALANCE AND CHECKING FOR NEW TRANSACTIONS
rsk.wallet.public.key=${RSK_WALLET_PUBLIC_KEY}
# RSK WALLET PRIVATE KEY FOR SENDING TRANSACTIONS TO RSK-BRIDGE-CONTRACT
rsk.wallet.private.key=${RSK_WALLET_PRIVATE_KEY}

# RSK BRIDGE CONTRACT FOR COINS TO BE SENT TO
# mainnet RSK Bridge Contract address: 0x0000000000000000000000000000000001000006
# https://developers.rsk.co/rsk/rbtc/conversion/networks/mainnet/
# testnet 0x0000000000000000000000000000000001000006
# https://developers.rsk.co/rsk/rbtc/conversion/networks/testnet/
rsk.bridge.address=${RSK_BRIDGE_ADDRESS} 

# BITCOIN NODE URL FOR MONITORING BTC CHAIN AND SENDING TRANSACTIONS
btc.service.url=${BTC_SERVICE_URL}
# USER/PW OR COOKIE VALUE FOR COMMUNICATION WITH BITCOIN NODE
btc.rpc.cookie=${$BTC_RPC_COOKIE}
# BITCOIN WALLET PUBLIC ADDRESS FOR MONITORING FOR NEW TRANSACTIONS
# IT SHOULD BE DERIVED FROM RSK_PUBLIC_KEY
# https://github.com/rsksmart/utils
# https://developers.rsk.co/rsk/rbtc/conversion/networks/mainnet/
btc.wallet.public.key=${RSK_WALLET_PUBLIC_KEY}
btc.wallet.private.key=${RSK_WALLET_PRIVATE_KEY}

# LIGHTNING WALLET ONCHAIN ADDRESS TO SEND FUNDS BEFORE LOOP IN
lnd.wallet.address=${LND_WALLET_ADDRESS}

# LND LOOP HTTP API ENDPOINT FOR INITIAING LOOPIN/LOOPOUT
lnd.loop.url=${LND_LOOP_URL}
# ADMIN MACAROON GENERATED BY LOOP CLIENT, IN HEX VALUE
# COMMAND: xxd -p -c2000 ~/.loop/testnet/loop.macaroon
# https://github.com/lightningnetwork/lnd/issues/2951
lnd.loop.admin.macaroon=${LND_LOOP_ADMIN_MACAROON}

```


## Helpful information
### RSK
- https://blockchain.oodles.io/dev-blog/implementing-ethereum-listener-for-listening-transactions/
- http://docs.web3j.io/4.8.7/advanced/filters_and_events/
- https://www.baeldung.com/web3j
### BTC
- https://bitcoinj.org/working-with-the-wallet#learning-about-changes
- https://bitcoin.stackexchange.com/questions/70063/how-do-i-parse-the-zeromq-messages-in-java