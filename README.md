<h1 align="center"> Gitopia Node Testneti </h1>


![image](https://user-images.githubusercontent.com/111747226/201535884-b2faca28-99a3-4508-8d85-375fc67a2d29.png)


### Notlar:


### Aşağıdaki rehberden digital ocean veya google cloud ile sunucuları kiralayıp kurabilirsiniz.
  
   # Daha önce Node kurulumu yapmadıysanız buradan sırasıyla adımları takip ederek tüm süreci öğrenebilirsiniz.
  ## Yeni Başladım Rehberi; [Pusula Finans Labs.](https://www.labs.pusulafinans.com/category/rehber/)
  ### 1. [Testnet ve Node test kurulum rehberi Bölüm-1](https://www.labs.pusulafinans.com/2022/08/23/testnet-ve-node-kurulum-rehberi/)
  ### 2. [Yeni Chrome Tarayıcı nasıl açarım?](https://www.labs.pusulafinans.com/2022/08/23/yeni-chrome-tarayici-nasil-acarim/)
  ### 3. [Ücretsiz Sunucu Kiralama](https://www.labs.pusulafinans.com/2022/08/23/nasil-ucretsiz-sunucu-kiralarim/)
  ### 4. [Digital Ocean Nasıl Kayıt olurum?](https://www.labs.pusulafinans.com/2022/08/23/digital-oceana-nasil-kayit-olabilirim/)
  ### 5. [MobaXTerm Terminal Kurulumu](https://www.labs.pusulafinans.com/2022/08/23/mobaxterm-terminal-kurulumu/)


 * Gitopia için nerede sohbet bu kanalda: [burada](https://t.me/pusulafinans) 
 * Gitopia discord kanalına girmeyi unutmayın ===> [burada](https://discord.com/invite/aqsKW3hUHD) 

<h1 align="center"> Node Kurulumu için adımları takip etmeyi unutmayın </h1>


### Sistem Gereksinimleri 

|CPU | RAM  | Disk  | 
|----|------|----------|
|   4| 8GB  | 200GB    |



## Sistem güncellemesi yapıyoruz
```
sudo apt update && sudo apt upgrade -y
```
## Gerekli kütüphanelerin kurulumunu yapıyoruz.
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

## Validator adınızı " " içinde yazın
```
MONIKER="PusulaFinans"
GITOPIA_CHAIN_ID="gitopia-janus-testnet-2"
```

## Go kurulumu:
```
cd $HOME
wget -O go1.18.4.linux-amd64.tar.gz https://golang.org/dl/go1.18.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && rm go1.18.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

## Gitopia portunu açalım:
```
PORT=15
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export GCHAIN_ID=gitopia-janus-testnet-2" >> $HOME/.bash_profile
echo "export GPORT=${GPORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binary dosyamızı yapılandırıyoruz ve kurulum yapıyoruz.
```
cd $HOME 
rm -rf gitopia
curl https://get.gitopia.com | bash
git clone -b v1.2.0 gitopia://gitopia/gitopia
cd gitopia 
make install
```
## Gitopia Versiyon kontrol ediyoruz

* version: 1.2.0
```
gitopiad version --long
```

## Başlatıyoruz:

* Bu kısımda bir işlem yapmanıza gerek yok!!

```
gitopiad init --chain-id "$GITOPIA_CHAIN_ID" "$MONIKER"
```
## Genesis ve addrbook'u indiriyoruz:
```
wget -O $HOME/.gitopia/config/addrbook.json "http://65.108.6.45:8000/gitopia/addrbook.json"
wget https://server.gitopia.com/raw/gitopia/testnets/master/gitopia-janus-testnet-2/genesis.json.gz
gunzip genesis.json.gz
mv genesis.json $HOME/.gitopia/config/genesis.json
```

## Seed ve Peers ayarlıyoruz:
```
SEEDS="399d4e19186577b04c23296c4f7ecc53e61080cb@seed.gitopia.com:26656"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gitopia/config/config.toml
```
## Disk yerimizi azaltıyoruz;
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.gitopia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gitopia/config/app.toml
```

## İndexer kapatmak için:
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gitopia/config/config.toml
```

## Gaz ve ücretleri optimasyonu:
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utlore\"/" $HOME/.gitopia/config/app.toml
```

## Prometheus etkinleştir:
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gitopia/config/config.toml
```

## Servis dosyasını oluşturalım:
```
sudo tee /etc/systemd/system/gitopiad.service > /dev/null <<EOF
[Unit]
Description=gitopia
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gitopiad) start --home $HOME/.gitopia
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Servis dosyasını yetkilendirip nodu başlatalım:

```
apt install screen
```

```
screen -S gitopia
```

## Kodları sırasıyla girelim:
```
sudo systemctl daemon-reload
sudo systemctl enable gitopiad
sudo systemctl restart gitopiad 
sudo journalctl -u gitopiad -f -o cat
```

## Yukarıda anlattığım eşleşme olduktan aşağıdaki komutu girdiğiniz `false` çıktısı vermeli

 * Eşleşmeden komutu girerseniz `true` yazar.
 * Node eşleşene kadar biz diğer işlemleri yapalım, cüzdan oluşturma kısmına geçin:

```
gitopiad status 2>&1 | jq .SyncInfo
```

## Cüzdan oluşturma: 

* cüzdanismi yazan kısmı kendınız belirleyin

```
gitopiad keys add cüzdanismi
```


## Şimdi test tokeni alacağız

 * Bunun için faucet botu değil, Platforma Keplr cüzdanı bağlayıp oradaki fauceti kullanacağız.
### Keplr Cüzdan Kurulumu;
 * Keplr wallet indirin ==> [burada](https://chrome.google.com/webstore/detail/keplr/dmkamcknogkgcdfhhbddcghachkejeap)
 * Import existing account seçeneğine basın.
 * Yukarıda oluşturduğunuz cüzdanının 12 ve 24 kelimesini buraya girip cüzdanı açın.
  *Buradan platforma giderek Keplr cüzdanınızı bağlayın. [Platform linki](https://gitopia.com/home)
 * Burada yeni bir profil oluşturun.
 * TLORE tuşuna basarak test tokeni alın.
 
## Cüzdan bakiyenizi kontrol etmek için:

* Cüzdan adresinizi girin
* Eğer nodunuz eşleşmediyse bu komutu girince balance 0 gözükür
* Ama keplrda tokenler gözükür, çünkü o güncel bloktadır.

```
gitopiad query bank balances cüzdanadresi
```

## Node eşleştiğinde validatorümüzü kuracağız.
* Eşleşene kadar bekliyorsun!!
* from cüzdanismi yazan yere, cüzdan adınız giriniz.
* Validatorismi yazan yere yukarıda belirlediğiniz validator isminizi yazın.

```
gitopiad tx staking create-validator \
  --amount 1000000utlore \
  --from cüzdanismi \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(gitopiad tendermint show-validator) \
  --moniker Validatorismi \
  --chain-id gitopia-janus-testnet-2
```

## Validatore stake etmek için:


```
gitopiad tx staking delegate validatöradresi 10000000utlore --from=cüzdanadresi --chain-id=$GITOPIA_CHAIN_ID --gas=auto
```

# Faydalı komutlar:


### Node'u silme
```
sudo systemctl stop gitopiad
sudo systemctl disable gitopiad
sudo rm /etc/systemd/system/gitopia* -rf
sudo rm $(which gitopiad) -rf
sudo rm $HOME/.gitopia* -rf
sudo rm $HOME/gitopia -rf
sed -i '/GITOPIA_/d' ~/.bash_profile
```

### Jailden çıkma:
```
gitopiad tx slashing unjail --from cüzdanismi --chain-id $GCHAIN_ID
```





## Ödül:

Okuyabilirsin: [Link](https://blog.gitopia.com/post/2022/11/the-janus-testnet-upgrade/index.html)





