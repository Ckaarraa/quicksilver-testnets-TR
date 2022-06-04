# Quicksilver Protocol'ün Rhapsody node kurulumu Türkçe rehberi

Ruesandora tarafından Türkçeleştirilmiştir.

![image](https://user-images.githubusercontent.com/101149671/172024390-3d83197f-e80a-469a-bd1f-35dd0710cb4d.png)

Rhapsody testnet, 1975 yılında Queen grubu için Freddie Mercury tarafından yazılan "Bohemian Rhapsody" adlı hit şarkının adını almıştır. Bu testnet'e katılırken ilham almak için lütfen bunu arka planda dinlemekten çekinmeyin: [YouTube'da Bohemian Rhapsody](https://www.youtube.com/watch?v=fJ9rUzIMcZQ ). :)

Tüm Quicksilver test ağları, Freddie Mercury ve / veya Queen'in şarkılarından sonra adlandırılacak, çünkü adamın lirik ve müzikal bir dehadan başka bir sebep yoktu ve Quicksilver -> Mercury -> Freddie arasında sömürü için olgunlaşmış biraz zayıf bir bağlantı var.

Yolculuğunuza yardımcı olmak için bir sürü komut dosyası ekledik; işletim sisteminize bağlı olacak bazı ek bağımlılıklarla birlikte `make`, `bash`, `git`, `jq`, `gcc` ve `go` (v1.17) yüklemeniz gerekir (örn.++').

Önümüzdeki günlerde ek görevler eklenecektir.

## Detaylar:

 - Chain-ID: `quicktest-3`
 - Launch Date: 2022-05-02
 - Current Version: `v0.1.10`
 - Genesis File: https://raw.githubusercontent.com/ingenuity-build/testnets/main/rhapsody/genesis.json

### Donanım Gereksinimleri:

Herhangi bir Cosmos-SDK zinciri gibi, donanım gereksinimleri de oldukça düşük:

 - 4x CPUs
 - 8GB RAM
 - 40GB Disk (statesync kullanılıyor, bu nedenle disk gereksinimleri düşüktür)
 - Kalıcı İnternet bağlantısı (testnet sırasında trafik minimum olacaktır; 10 Mbps yeterli olacaktır - üretim için en az 100 Mbps beklenmektedir)

### Node:

Aşağıdaki düğümleri çalıştırıyoruz:

 - node01.quicktest-1.quicksilver.zone:26657
 - node02.quicktest-1.quicksilver.zone:26657
 - node03.quicktest-1.quicksilver.zone:26657
 - node04.quicktest-1.quicksilver.zone:26657

Seeds:

 - dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.quicktest-1.quicksilver.zone:26656


## Yarı Otomatik Yapılandırma

```
## clone this repo
git clone https://github.com/ingenuity-build/testnets ## this repo
cd testnets/rhapsody

## download and build quicksilverd and gaiad
make init

## show keys
make keys

## follow instructions listed to get funds from the faucet via discord

## check balances
make balances

## start the validator
make start

## view the logs
make logs

## submit a create-validator tx to start validating (enter your validator name when prompted)
make validate 

---

## clean up time! (post-testnet)
make stop 

make clean
```

### Neyi yanlış yapıyorum?!

#### Finanse edilmeyen hesap
```
joe@desktop:~/code/testnets/rhapsody$ make validate
Enter your validator name: my_validator
Error: rpc error: code = NotFound desc = rpc error: code = NotFound desc = account quick1fk9qtycszzk32c3hk8xwjwvkhmkc8rv6gg0xzd not found: key not found
```

Çözüm: Adresinize para yatırmak için discord #qck-tap kanalını kullanın (gelmesi birkaç saniye sürebilir!)

#### Düğüm çalışmıyor
```
joe@desktop:~/code/testnets/rhapsody$ make validate
Enter your validator name: my_validator
Error: post failed: Post "http://localhost:26657": dial tcp 127.0.0.1:26657: connect: connection refused
...
```

Çözüm: Düğümünüz çalışmıyor; `make start`. komutunu çalıştırın. Sorun devam ederse, `make logs` bir göz atın ve discord'da birini bulun!

## Manuel Yapılandırma

Quicksilver'ı indirin ve oluşturun:

    git clone https://github.com/ingenuity-build/quicksilver.git --branch v0.1.10
    cd quicksilver
    make build

Testnet komut dosyası script (`touch scripts/testnet_conf.sh`):

    #!/bin/bash -i
    
    set -xe
    
    ### CONFIGURATION ###
    
    CHAIN_ID=quicktest-3
    
    GENESIS_URL="https://raw.githubusercontent.com/ingenuity-build/testnets/main/rhapsody/genesis.json"
    SEEDS="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.quicktest-1.quicksilver.zone:26656"
    
    BINARY=./build/quicksilverd
    NODE_HOME=$HOME/.quicksilverd
    
    # SET this value for your node:
    NODE_MONIKER="Your_Node"
    
    ### STATE SYNC ###
    # To sync the chain on v0.1.10, you _will_ need to use statesync. See testnets/rhapsody/quicksilver.sh for more information.
    
    # if you set this to true, please have TRUST HEIGHT and TRUST HASH and RPC configured
    export STATE_SYNC=false
    # set height
    export TRUST_HEIGHT=
    # set hash
    export TRUST_HASH=""
    export SYNC_RPC="http://node02.quicktest-1.quicksilver.zone:26657,http://node03.quicktest-1.quicksilver.zone:26657,http://node04.quicktest-1.quicksilver.zone:26657"
    
    echo  "Initializing $CHAIN_ID..."
    $BINARY config chain-id $CHAIN_ID --home $NODE_HOME
    $BINARY config keyring-backend test --home $NODE_HOME
    $BINARY config broadcast-mode block --home $NODE_HOME
    $BINARY init $NODE_MONIKER --chain-id $CHAIN_ID --home $NODE_HOME
    
    echo "Get genesis file..."
    curl -sSL $GENESIS_URL > $NODE_HOME/config/genesis.json
    
    if  $STATE_SYNC; then
        echo  "Enabling state sync..."
        sed -i -e '/enable =/ s/= .*/= true/'  $NODE_HOME/config/config.toml
        sed -i -e "/trust_height =/ s/= .*/= $TRUST_HEIGHT/"  $NODE_HOME/config/config.toml
        sed -i -e "/trust_hash =/ s/= .*/= \"$TRUST_HASH\"/"  $NODE_HOME/config/config.toml
        sed -i -e "/rpc_servers =/ s/= .*/= \"$SYNC_RPC\"/"  $NODE_HOME/config/config.toml
    else
        echo  "Disabling state sync..."
    fi
    
    echo "Set seeds..."
    sed -i -e "/seeds =/ s/= .*/= \"$SEEDS\"/"  $NODE_HOME/config/config.toml

Quicksilver deposu ana dizininden bu betiği çalıştırın;

Yürütülebilir hale getirmeyi unutmayın:

    chmod +x scripts/testnet_conf.sh

Sonra basitçe çalıştırın:

    ./scripts/testnet_conf.sh

## Düğümünüzü çalıştırma
At this point you can run the node on the CLI with `./build/quicksilverd start` to ensure everything is configured correctly. At this point you may configure your system to run Quicksilver as a system service or daemon.

## Validatör kurulum

### Test Wallet
Doğrulayıcı olarak çalışmak için bir QCK cüzdanı oluşturmanız gerekir:

    ./build/quicksilverd keys add $YOUR_TEST_WALLET --keyring-backend=test

Halihazırda bir test cüzdanınız varsa, run kullanmak (ve anımsatıcınızı girmek) istersiniz:

    ./build/quicksilverd keys add $YOUR_TEST_WALLET --recover --keyring-backend=test

### Faucet

QCK ve ATOM musluklarına erişmek için discord sunucumuza katılın. Uygun kanalda olduğunuzdan emin olun:

 - **qck-tap** for QCK tokens;
 - **atom-tap** for ATOM tokens;

Musluk adresini kontrol etmek için:

    $faucet_address rhapsody

Bakiyenizi kontrol etmek için:

    $balance $YOUR_TEST_WALLET rhapsody

Musluk hibesi talep etmek için:

    $request $YOUR_TEST_WALLET rhapsody

### Validator Tx

Ardından, doğrulayıcı durumuna yükseltmek için tx'i çalıştırmanız yeterlidir:

    ## Upgrade node to validator
    ./build/quicksilverd tx staking create-validator \
      --from=$YOUR_TEST_WALLET \
      --amount=1000000uqck \
      --moniker=$NODE_MONIKER \
      --chain-id=$CHAIN_ID \
      --commission-rate=0.1 \
      --commission-max-rate=0.5 \
      --commission-max-change-rate=0.1 \
      --min-self-delegation=1 \
      --pubkey=$($BINARY tendermint show-validator)


## Quicksilver'da qAtoms basımının kullanılması

Talimatlar önümüzdeki hafta erken gelecek!

# Hesaplar:

[Twitter](https://twitter.com/Ruesandora0)

[Forum](https://forum.rues.info/index.php)

[Telegram Announcement](https://t.me/RuesAnnouncement)

[Telegram Chat](https://t.me/RuesChat)

[Telegram Node](https://t.me/RuesNode)

[Telegram Node Chat](https://t.me/RuesNodeChat)
