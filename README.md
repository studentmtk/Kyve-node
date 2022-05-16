# Kyve-node

Требования к серверу:
1 CPU  4 GB RAM  40 GB SSD - Ubuntu 20.04

В начале вам нужно получить небольшоое количество токенов AR, для этого вам понадобится твиттер аккаунт. Перейдите на  [сайт](https://faucet.arweave.net/), проставьте галки, скачайте json файл, переименуйте его в "arweave.json", нажмите на "Open tweet pop-up", опубликуйте твит и через некоторое время вы получите небольше количество токенов AR. Затем перенесите этот файл к себе на сервер с помощью scp или filezilla в домашнюю папку.

Установка оркужения:
```
sudo apt-get update -y; sudo apt-get upgrade -y; sudo apt install unzip -y
```
 
Создание папки kyve
```
mkdir $HOME/kyve; cd $HOME/kyve
```
Скачиваем бинарные файлы

```
wget -O evm-linux.zip https://github.com/KYVENetwork/evm/releases/download/v1.0.5/evm-linux.zip
wget -O kyve-bitcoin-linux.zip https://github.com/kyve-org/bitcoin/releases/download/v0.0.0/kyve-bitcoin-linux.zip
wget -O kyve-solana-linux.zip https://github.com/kyve-org/solana/releases/download/v0.0.1/kyve-solana-linux.zip
wget -O kyve-zilliqa-linux.zip https://github.com/kyve-org/zilliqa/releases/download/v0.0.0/kyve-zilliqa-linux.zip
wget -O stacks-linux.zip https://github.com/kyve-org/stacks/releases/download/v0.0.2/stacks-linux.zip
wget -O celo-linux.zip https://github.com/kyve-org/celo/releases/download/v0.0.0/kyve-celo-linux.zip
wget -O near-linux.zip https://github.com/kyve-org/near/releases/download/v0.0.1/kyve-near-linux.zip
wget -O kyve-evmos-linux.zip https://github.com/kyve-org/evm/releases/download/v1.0.5/kyve-evm-linux.zip

```
Распаковываем архивы, делаем файлы исполняемыми и переносим их в папку /usr/local/bin/

```
unzip -o "*.zip"
chmod +x evm-linux kyve-solana-linux kyve-zilliqa-linux bitcoin-linux stacks-linux kyve-celo-linux  kyve-near-linux kyve-evm-linux
mv evm-linux kyve-solana-linux kyve-zilliqa-linux bitcoin-linux stacks-linux kyve-near-linux kyve-evm-linux  kyve-celo-linux /usr/local/bin/

```
Создаем сервисные файлы для каждого пула
```
MNEMONIC="указываем вашу сид фразу (мнемонику )"
```
```
STAKE="указываем количество токенов, которое вы хотите застейкать"
```
пул Moonbeam
```
echo "[Unit]
Description=Moonbeam
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evm-linux) --poolId 0 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535
unzip -o "*.zip"
[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/moonbeam.service

```

пул Avalanche

```
echo "[Unit]
Description=Avalanche
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evm-linux) --poolId 1 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/avalanche.service


```
пул Bitcoin

```
echo "[Unit]
Description=Bitcoin
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which bitcoin-linux) --poolId 3 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/bitcoin.service

```

пул Solana

```
echo "[Unit]
Description=Solana
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which kyve-solana-linux) --poolId 4 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/solana.service

```

пул Zilliqa

```
[Service]
User=$USER
Type=simple
ExecStart=$(which kyve-zilliqa-linux) --poolId 5 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/zilliqa.service

```

пул Near
```
echo "[Unit]
Description=near
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which kyve-near-linux) --poolId 6 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/near.service

```

пул Celo

```
echo "[Unit]
Description=celo
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which kyve-celo-linux) --poolId 7 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/celo.service

```

пул Evmos
```
echo "[Unit]
Description=evmos
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evm-linux) --poolId 8 --mnemonic \"$MNEMONIC\" --keyfile $HOME/arweave.json --initialStake $STAKE
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/kyve/service/evmos.service

```

Перенесем наши сервисные файлы в папку /etc/systemd/system/
```
mv $HOME/kyve/service/* /etc/systemd/system/
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
sudo systemctl daemon-reload
sudo systemctl restart systemd-journald
```

Запуск пула ( при необходимости с новым значение токенов для стейка ) на примере Avalanche

```
STAKE=NEW_STAKE
sed -i.bak -e "s/initialStake .*/initialStake $STAKE/" /etc/systemd/system/avalanche.service
systemctl daemon-reload
systemctl restart avalanche

```
просмотр логов

```
journalctl -u avalanche -f

```
Помните, что минимальное значение количества токенов для бонда в пулах постоянно меняется. В каждом пуле может быть не более 50 валидаторов. Если вы выпадаете из этого числа, то ваши застейканные тонкены автоматически расстейкиваются. В данный момент попасть в активный сет валидаторов достаточно сложно.

















