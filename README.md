## IBC-relayer-for-KiChain (KiChain + IBC Relayer <--> Umee)  
### In addition to the Ki node, you will need a Umee node on another server.  
Change to `laddr = "tcp://127.0.0.1:26657"` in `/root/kichain/kid/config/config.toml` and restart the service  

```
sudo systemctl restart kichaind
```  
## 
### Install the relayer to KiChain Server  
```
wget https://github.com/cosmos/relayer/releases/download/v0.9.3/Cosmos.Relayer_0.9.3_linux_amd64.tar.gz
```  
```
tar -zxvf Cosmos.Relayer_0.9.3_linux_amd64.tar.gz
```  
```
cp "Cosmos Relayer" /usr/local/bin/rly
```  
### Initialize the relayer's configuration  
```
rly config init
```  
### Manually edit `/root/.relayer/config/config.yaml`  
```
global:
  api-listen-addr: :5183
  timeout: 3m
  light-cache-size: 20
chains:
- key:
  chain-id: umee-betanet-1
  rpc-addr: http://`<your_address_umee_node>:<umee_port>`
  account-prefix: umee
  gas-adjustment: 1.5
  gas-prices: 0.025uumee
  trusting-period: 10m
- key:
  chain-id: kichain-t-4
  rpc-addr: http://127.0.0.1:26657
  account-prefix: tki
  gas-adjustment: 1.5
  gas-prices: 0.025utki
  trusting-period: 10m
paths: {}
```  
### Create umee key  
```
rly keys add umee-betanet-1 umeekey
```
Output:  
> "mnemonic":"<your_mnemonic>":"umee<your_umee_wallet>"  

### Restore your KiChain node key  
```
rly keys restore kichain-t-4 kikey "<paste_your_mnemonic>"
```  
Output:  
> "<your_tki_wallet>"  

### Add keys to chains  
```
rly chains edit umee-betanet-1 key umeekey
```
```
rly chains edit kichain-t-4 key kikey
```  
##  
### Go to umee discord server and request tokens from the faucet. And Check balances  
```
rly query balance umee-betanet-1
```
```
rly query balance kichain-t-4
```
Output:  
> root@vmi1:~# rly query balance umee-betanet-1  
> 100000000uumee  
> root@vmi1:~# rly query balance kichain-t-4  
> 10000000utki  
##  
### Initialize the light clients  
```
rly light init umee-betanet-1 -f
```
```
rly light init kichain-t-4 -f
```  
Output:
> root@vmi1:~# rly light init umee-betanet-1 -f  
successfully created light client for umee-betanet-1 by trusting endpoint http://<your_address_umee_node>:<umee_port>...  
root@vmi1:~# rly light init kichain-t-4 -f  
successfully created light client for kichain-t-4 by trusting endpoint http://127.0.0.1:26657...  
##  
### Create path umee_to_ki  
```
rly paths generate umee-betanet-1 kichain-t-4 umee_to_ki_path --port=transfer
```
Output:  
> root@vmi1:~# rly paths generate umee-betanet-1 kichain-t-4 umee_to_ki_path --port=transfer  
Generated path(umee_to_ki_path), run 'rly paths show umee_to_ki_path --yaml' to see details  

### Check chains  
```
rly chains list
```
Output:  
> root@vmi1:~# rly chains list  
 0: umee-betanet-1       -> key(✔) bal(✔) light(✔) path(✔)  
 1: kichain-t-4          -> key(✔) bal(✔) light(✔) path(✔)  
 
 ### Open channel umee_to_ki and wait for the channel to be created  
 ```
 rly tx link umee_to_ki_path
 ```  
 Output:  
 > root@vmi1:~# rly tx link umee_to_ki_path  
I[2021-09-12|16:53:18.324] ★ Clients created: client(07-tendermint-9) on chain[umee-betanet-1] and client(07-tendermint-325) on chain[kichain-t-4]  
I[2021-09-12|16:53:18.509] ★ Connection created: [umee-betanet-1]client{07-tendermint-9}conn{connection-78} -> [kichain-t-4]client{07-tendermint-325}conn{connection-335}  
I[2021-09-12|16:53:18.662] ★ Channel created: [umee-betanet-1]chan{channel-0}port{transfer} -> [kichain-t-4]chan{channel-61}port{transfer}  

### Send tokens umee to ki  
```
rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee <your_tki_wallet> --path umee_to_ki_path
```
### Check balance  
```
rly query balance kichain-t-4
```  

### If the tokens have not arrived, manually open `/root/.relayer/config/config.yaml` and edit the lines:  
> paths:  
  umee_to_ki_path:  
    src:  
> ...  
> channel-id: channel-0  
> ...  
> dst:  
> channel-id: channel-61  

Then again open channel, send tokens, check balance:  
```
rly tx link umee_to_ki_path  
rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee <your_tki_wallet> --path umee_to_ki_path  
rly query balance kichain-t-4  
```
Output:  
> root@vmi1:~# rly tx link umee_to_ki_path  
I[2021-09-12|16:53:18.324] ★ Clients created: client(07-tendermint-9) on chain[umee-betanet-1] and client(07-tendermint-325) on chain[kichain-t-4]  
I[2021-09-12|16:53:18.509] ★ Connection created: [umee-betanet-1]client{07-tendermint-9}conn{connection-78} -> [kichain-t-4]client{07-tendermint-325}conn{connection-335}  
I[2021-09-12|16:53:18.662] ★ Channel created: [umee-betanet-1]chan{channel-0}port{transfer} -> [kichain-t-4]chan{channel-61}port{transfer}  
root@vmi1:~# rly tx transfer umee-betanet-1 kichain-t-4 1000000uumee tki19au92d9gv0wclccayheyym98peagrwysf5pyej --path umee_to_ki_path  
I[2021-09-12|16:53:41.929] v [umee-betanet-1]@{285261} - msg(0:transfer) hash(AC7FC7B0E732955FF5FCDBE1146B75287ACFF42660422064E31F436E9C851355)  
root@vmi1:~# rly query balance kichain-t-4  
1000000transfer/channel-61/uumee,10000000utki  
##  
### Create path ki_to_umi  
```
rly paths generate kichain-t-4 umee-betanet-1 ki_to_umee_path --port=transfer
```  
### Open channel ki_to_umee and wait for the channel to be created  
```
rly tx link ki_to_umee_path
```  
### Send tokens and check balance  
```
rly tx transfer kichain-t-4 umee-betanet-1 5000000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
rly query balance umee-betanet-1
```  
### If the tokens have not arrived, manually open `/root/.relayer/config/config.yaml` and edit the lines:  
> paths:  
  ki_to_umee_path:  
    src:  
> ...  
> channel-id: channel-61  
> ...  
> dst:  
> channel-id: channel-0  

Then again open channel, send tokens `(5 times)`, check balance:  
```
rly tx link ki_to_umee_path  
rly tx transfer kichain-t-4 umee-betanet-1 5000000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
rly query balance umee-betanet-1  
```
Output:  
> root@vmi1:~# rly tx transfer kichain-t-4 umee-betanet-1 5000000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
I[2021-09-12|17:50:19.219] v [kichain-t-4]@{299734} - msg(0:transfer) hash(EB368083149FCFD44109BC77BF1AD89E7AA866B2ADE425423C84AC9877C4BFA2)  
root@vmi1:~# rly tx transfer kichain-t-4 umee-betanet-1 2500000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
I[2021-09-12|17:54:19.758] v [kichain-t-4]@{299772} - msg(0:transfer) hash(F347BF485F7935756DA5FCB05D78DB248F6C57EE4829B5547B9D44AED1F8963F)  
root@vmi1:~# rly tx transfer kichain-t-4 umee-betanet-1 2000000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
I[2021-09-12|17:54:46.279] v [kichain-t-4]@{299776} - msg(0:transfer) hash(6D7629D54BAF98F7FF92ADA01BB9291206929EB2E3D496676FD8EB9A49F39E92)  
root@vmi1:~# rly tx transfer kichain-t-4 umee-betanet-1 1500000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
I[2021-09-12|17:55:35.551] v [kichain-t-4]@{299784} - msg(0:transfer) hash(A3E8DDE57B80260FB63CD06B6E29149B7C61640AAD7BB1143C82F0B59EB317F9)  
root@vmi1:~# rly tx transfer kichain-t-4 umee-betanet-1 1000000utki umee1uflrxeum6rk3vhnses7hsh7ypndvlkzguswvuu --path ki_to_umee_path  
I[2021-09-12|17:56:05.996] v [kichain-t-4]@{299789} - msg(0:transfer) hash(9CBDC8FAB6447CE86CE9E0845A5ACD19FCD7336251FF8FBFBDDCFF58F201DC2A)  
root@vmi1:~# rly query balance umee-betanet-1  
17000000transfer/channel-0/utki,97808820uumee  
##  
### That's all. Good luck.
