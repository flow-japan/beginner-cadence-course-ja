# Глава 6 День 1 - Создание Testnet аккаунта и деплой в Testnet

Привет, ботаники! В сегодняшнем уроке мы узнаем, как создать новый аккаунт testnet и развернуть наш контракт NFT в Flow Testnet.

## Установка расширения Cadence через VSCode Extension

*Если вы еще не установили VSCode, вы можете сделать это здесь: https://code.visualstudio.com/*

Теперь, когда мы больше не на игровой площадке, мы хотим иметь возможность отображать ошибки в VSCode при программировании в Cadence. Для этого существует расширение!

> Откройте VSCode. В левой части VSCode есть значок, похожий на 4 квадрата. Нажмите на него и найдите "Cadence".

> Нажмите на расширение и нажмите "Установить":

<img src="../images/cadence-vscode-extension.png" />

## Установка Flow CLI и flow.json

Flow CLI позволит нам запускать транзакции и скрипты из терминала, а также позволит нам выполнять другие действия с Flow, например, развернуть контракт.

> Установите [Flow CLI](https://docs.onflow.org/flow-cli/install/). Это можно сделать следующим образом:

**Mac**
- Вставьте `sh -ci "$(curl -fsSL https://storage.googleapis.com/flow-cli/install.sh)"` в терминал

**Windows**
- Вставьте `iex "& { { $(irm 'https://storage.googleapis.com/flow-cli/install.ps1') }"` в PowerShell

**Linux** 
- Вставьте `sh -ci "$(curl -fsSL https://storage.googleapis.com/flow-cli/install.sh)"` в терминал

Вы можете убедиться, что Flow CLI установлен, зайдя в терминал и набрав `flow version`. Если версия появится, значит, все готово.

## Flow папка

Внутри нашего базового каталога создадим новую папку `flow`.

Внутри папки `flow` создадим еще одну папку `cadence`.

Внутри папки `cadence` создадим папку `contracts`, папку `transactions` и папку `scripts`.

Внутри папки `contracts` добавьте новый файл `CONTRACT_NAME.cdc`. Замените CONTRACT_NAME на название вашего контракта. В этот файл поместите код вашего контракта из главы 5.

Обратите внимание, что теперь нам нужно импортировать из локального пути к файлу, а не из случайного адреса Flow playground. Мы больше не импортируем из `0x01`, это была просто фишка игровой площадки. В данном случае мы импортируем из локального контракта, который существует в нашем проекте.

> Измените импорт в верхней части так: `import NonFungibleToken from "./NonFungibleToken.cdc"`.

Чтобы это сработало, нам также нужно добавить интерфейс контракта `NonFungibleToken` в папку `contracts`. Не забудьте назвать файл `NonFungibleToken.cdc`.

---

Внутри папки транзакций создайте несколько файлов с именем `TRANSACTION_NAME.cdc`. Замените TRANSACTION_NAME на названия ваших транзакций.

Обратите внимание, что импорты теперь тоже неправильные. Мы больше не импортируем из `0x01`, это была просто игровая площадка. В данном случае мы импортируем из локального контракта, который существует в нашем проекте. Поэтому измените импорт на что-то вроде этого:

```cadence
import ExampleNFT from "../contracts/ExampleNFT.cdc"
```

--- 

Внутри папки scripts создайте несколько файлов с именем `SCRIPT_NAME.cdc`. Замените SCRIPT_NAME на названия ваших скриптов.

---

### flow.json

> Теперь, когда у нас есть наш контракт в каталоге проекта, перейдите в терминал и напишите `cd` чтобы перейти в каталог базового проекта. 

> Введите `flow init`.

Это создаст файл `flow.json` в вашем проекте. Он необходим для развертывания контрактов и для выдачи ошибок компиляции в нашем коде Cadence.

## Развертывание нашего контракта NFT в TestNet

Отлично! Теперь давайте развернем наш контракт в TestNet, чтобы мы могли начать взаимодействовать с ним.

## Настройка `flow.json`

> Внутри файла `flow.json` сделайте так, чтобы объект "contracts" выглядел следующим образом:

```json
"contracts": {
  "ExampleNFT": "./contracts/ExampleNFT.cdc",
  "NonFungibleToken": {
    "source": "./contracts/NonFungibleToken.cdc",
    "aliases": {
      "testnet": "0x631e88ae7f1d7c20"
    }
  }
},
```

Это позволит вашему `flow.json` знать, где находятся ваши контракты. Обратите внимание, что `NonFungibleToken` уже существует в Flow Testnet, поэтому он выглядит сложнее.

### Создание аккаунта

> 🔐 Сгенерируйте адрес **деплойера**, введя в терминале команду `flow keys generate --network=testnet`. Обязательно сохраните где-нибудь свой открытый ключ и закрытый ключ, они вам скоро понадобятся.

<img src="https://i.imgur.com/HbF4C73.png" alt="generate key pair" />

> 👛 Создайте учетную запись **деплойера**, перейдя по адресу https://testnet-faucet.onflow.org/, вставьте свой открытый ключ, указанный выше, и нажмите `CREATE ACCOUNT`: 

<img src="https://i.imgur.com/73OjT3K.png" alt="configure testnet account on the website" />

> По окончании нажмите `COPY ADDRESS` и обязательно сохраните этот адрес. Он вам понадобится!

> ⛽️ Добавьте ваш новый аккаунт testnet в ваш `flow.json`, изменив следующие строки кода. Вставьте ваш адрес, который вы скопировали выше, туда, где написано "YOUR GENERATED ADDRESS", и вставьте ваш приватный ключ туда, где написано "YOUR PRIVATE KEY".

```json
"accounts": {
  "emulator-account": {
    "address": "f8d6e0586b0a20c7",
    "key": "5112883de06b9576af62b9aafa7ead685fb7fb46c495039b1a83649d61bff97c"
  },
  "testnet-account": {
    "address": "YOUR GENERATED ADDRESS",
    "key": {
      "type": "hex",
      "index": 0,
      "signatureAlgorithm": "ECDSA_P256",
      "hashAlgorithm": "SHA3_256",
      "privateKey": "YOUR PRIVATE KEY"
    }
  }
},
"deployments": {
  "testnet": {
    "testnet-account": [
      "ExampleNFT"
    ]
  }
}
```

> 🚀 Разверните свой смарт-контракт ExampleNFT:

```sh
flow project deploy --network=testnet
```

<img src="../images/deploy-contract.png" alt="deploy contract to testnet" />

## Квесты

1. Перейдите на сайт https://flow-view-source.com/testnet/. Там, где написано "Account", вставьте сгенерированный вами адрес Flow и нажмите "Go". С левой стороны вы должны увидеть ваш контракт NFT. Разве не здорово видеть его в прямом эфире в Testnet? Затем отправьте URL-адрес страницы. 
- ПРИМЕР: https://flow-view-source.com/testnet/account/0x90250c4359cebac7/