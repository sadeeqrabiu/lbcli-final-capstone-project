# My Capston Walkthough

Hey reviewers! Here is my step by stp process on how I complted the final task. Had a ton of fun doing this!

## Createing the Wallet
first i created the wallet using my github username as instructed:
`bitcoin-cli -signet createwallet "sadeeqrabiu" false false "" false true`

Then I exported my descriptor by lookign through the json output for the wpkh array:
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu listdescriptors`

## Testnet Address and Faucet
getting an address was simple:
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu getnewaddress "" "bech32"` 
-> tb1qr9rjkjveuzhx2zwjp3r26hd7tjtessuslvaftq

i pasted it into the btrust alt faucet and boom, the funds arrived eventually!
transaction was mined in block `0000000557cf0825fdcc06a66847ee473b9c9d177a53a7861c70988992bb3479`.
I used `getblock <hash> 1` to fetch the first transaction in the tx array for `coinbase-1.txt`

## The 10k sats spend with precise fee
i had to be precise here and used createrawtransaction so i can set the fee exactly instead of relying on estimation!
first got change addres: `bitcoin-cli -signet -rpcwallet=sadeeqrabiu getrawchangeaddress` 
then manually calculated the diffs (103364 sats - 10000 sats - 700 sats) = 92664 sats back to me.
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu createrawtransaction '[{"txid":"...","vout":1407,"sequence":1}]' '{"tb1qddpcyus3u603n63lk7m5epjllgexc24vj5ltp7":0.00010000, "mychangeaddress":0.00092664}'`
I finalized and sent it and recorded `block-2`. Sequence 1 means RBF is enabled by defaut

## Multisig Wallets
I geenerated three new address and extracted the PubeKeys:
`pk=$(bitcoin-cli -signet -rpcwallet=sadeeqrabiu getaddressinfo <addr> | jq -r '.pubkey')` (did this 3 times)
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu createmultisig 2 '["pk1", "pk2", "pk3"]' "legacy"`
the redeem scrtipt and address was printed directy for me!

did the exact same thing for the native segwit P2WSH addresses but used `"bech32"` format and saved the witnessScript properly. 

## PSBT and Op_Return payload
For psbt it was super cool, i just used walletcreatefundedpsbt:
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu walletcreatefundedpsbt "[]" '[{"tb1q..":0.00005}]'`
then `walletprocesspsbt` and `finalizepsbt` to grab the hex and push it!

finally for the op_return, i converted 'sadeeqrabiu' to hex format (it gave me `7361646565717261626975`)
and created an empty output raw transaction containing the custom data variable!
`bitcoin-cli -signet -rpcwallet=sadeeqrabiu createrawtransaction "[]" '{"data":"7361646565717261626975"}'`
I funded it via `fundrawtransaction`, siged it with wallet and broadcasted! All done.

Thanks for the awesme course guys!
