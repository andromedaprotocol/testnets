# How to submit a governance proposal

#### The following guide will show you required steps in order to change the governance module parameter of minimum deposit amount for a proposal.

1. Create a `proposal.json` file with the appropriate contents. Here is an example
```json lines
{
   "title":"Deposit amount",
   "description":"Decrease the minimum deposit amount for gov proposals. Since this is a testnet, lets make this cheaper",
   "changes":[
      {
         "subspace":"gov",
         "key":"deposit_params",
         "value":"{\"min_deposit\":[{\"denom\":\"uandr\", \"amount\":\"5000000\"}]}"
      }
   ],
   "deposit":"10000000uandr"
}
```
2. Execute the following command: `andromedad tx gov submit-proposal param-change proposal.json --chain-id $CHAIN_ID --from $FROM_KEY_NAME --fees 250uandr`
   1. Adjust `$CHAIN_ID`, `$FROM_KEY_NAME` to suit your needs.
3. Sign the transaction and make sure that the result is successful. 

```text
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 2ADCF7D57235F79F5C66B9C81B7399CBE1D7292DCA6FC8B4206437F937F3120F
```
4. Check the transaction hash `2ADCF7D57235F79F5C66B9C81B7399CBE1D7292DCA6FC8B4206437F937F3120F` in a block explorer to verify its success. [Ping Explorer](https://ping.wildsage.io/andromeda)

##What can go wrong
1. Insufficient funds in wallet for the proposal
```text
code: 5
codespace: sdk
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '0uandr is smaller than 250uandr: insufficient funds: insufficient funds'
timestamp: ""
tx: null
txhash: 2ADCF7D57235F79F5C66B9C81B7399CBE1D7292DCA6FC8B4206437F937F3120F
```
