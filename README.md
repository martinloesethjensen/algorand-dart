<p align="center"> 
<img src="https://miro.medium.com/max/700/1*BFpFCJepifaREIg7qLSLag.jpeg">
</p>

# algorand-dart
[![pub.dev][pub-dev-shield]][pub-dev-url]
[![Effective Dart][effective-dart-shield]][effective-dart-url]
[![Stars][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

Algorand is a public blockchain and protocol that aims to deliver decentralization, scale and security for all participants.
Their PURE PROOF OF STAKE™ consensus mechanism ensures full participation, protection, and speed within a truly decentralized network. With blocks finalized in seconds, Algorand’s transaction throughput is on par with large payment and financial networks. And Algorand is the first blockchain to provide immediate transaction finality. No forking. No uncertainty.


## Introduction
Algorand-dart is a community SDK with an elegant approach to connect your Dart & Flutter applications to the Algorand blockchain, send transactions, create assets and query the indexer with just a few lines of code.

Once installed, you can simply connect your application to the blockchain and start sending payments

```dart
algorand.sendPayment(
    account: account,
    recipient: newAccount.address,
    amount: Algo.toMicroAlgos(5),
);
```

or create a new asset:

```dart
algorand.assetManager.createAsset(
    account: account,
    assetName: 'FlutterCoin',
    unitName: 'Flutter',
    totalAssets: 10000,
    decimals: 2,
);
```

## Features
* Algod
* Indexer
* Transactions
* Atomic Transfers
* Account management
* Asset management
* TEAL compilation
* Flutter 2.0 support :heart:

## Getting started

### Installation

You can install the package via pub.dev:

```yaml
algorand_dart: ^0.0.2-beta.3
```

> **Note**: Algorand-dart requires Dart >=2.12.0 & null safety
> See the latest version on pub.dev

## Usage
Create an ```AlgodClient``` and ```IndexerClient``` and pass them to the ```Algorand``` constructor.
We added extra support for locally hosted nodes & third party services (like PureStake).

```dart
final algodClient = AlgodClient(
    apiUrl: PureStake.TESTNET_ALGOD_API_URL,
    apiKey: apiKey,
    tokenKey: PureStake.API_TOKEN_HEADER,
);

final indexerClient = IndexerClient(
    apiUrl: PureStake.TESTNET_INDEXER_API_URL,
    apiKey: apiKey,
    tokenKey: PureStake.API_TOKEN_HEADER,
);

final algorand = Algorand(
    algodClient: algodClient,
    indexerClient: indexerClient,
);
```

## Account Management
Accounts are entities on the Algorand blockchain associated with specific onchain data, like a balance. An Algorand Address is the identifier for an Algorand account.

### Creating a new account

Creating a new account is as easy as calling:
```dart
final account = await algorand.createAccount();
```

Or you can always use the ```Account``` class.
```dart
final account = await Account.random();
```

With the given account, you can easily extract the public Algorand address, signing keys and seedphrase/mnemonic.
```dart
final publicAddress = account.publicAddress;
final words = await account.seedPhrase;
```

### Loading an existing account

You can load an existing account using your **generated secret key or binary seed**.

```dart
final account = await algorand.loadAccountFromSeed(seed);
```

### Restoring an account

Recovering an account from your 25-word mnemonic/seedphrase can be done by passing an **array or space delimited string**

```dart
final restoredAccount = await algorand.restoreAccount([/* 25 words */]);
```

## Transactions
There are multiple ways to create a transaction. We've included helper functions to make our life easier.

```dart
algorand.sendPayment(
    account: account,
    recipient: newAccount.address,
    amount: Algo.toMicroAlgos(5),
    note: 'Hi from Flutter!',
);
```

This will broadcast the transaction and immediately returns the transaction id, however you can also wait until the transaction is confirmed in a block using:

```dart
final transactionId = await algorand.sendPayment(
    account: account,
    recipient: newAccount.address,
    amount: Algo.toMicroAlgos(5),
    note: 'Hi from Flutter!',
    waitForConfirmation: true,
    timeout: 3,
);
```


Or you can use the ```TransactionBuilder``` to create more specific, raw transactions:

```dart
// Fetch the suggested transaction params
final params = await algorand.getSuggestedTransactionParams();

// Build the transaction
final transaction = await (PaymentTransactionBuilder()
    ..sender = account.address
    ..note = 'Hi from Flutter'
    ..amount = Algo.toMicroAlgos(5)
    ..receiver = recipient
    ..suggestedParams = params)
  .build();

// Sign the transaction
final signedTx = await transaction.sign(account);

// Send the transaction
final txId = await algorand.sendTransaction(signedTx);
```

## Atomic Transfer
An Atomic Transfer means that transactions that are part of the transfer either all succeed or all fail.
Atomic transfers allow complete strangers to trade assets without the need for a trusted intermediary,
all while guaranteeing that each party will receive what they agreed to.

Atomic transfers enable use cases such as:

* **Circular trades** - Alice pays Bob if and only if Bob pays Claire if and only if Claire pays Alice.
* **Group payments** - Everyone pays or no one pays.
* **Decentralized exchanges** - Trade one asset for another without going through a centralized exchange.
* **Distributed payments** - Payments to multiple recipients.

An atomic transfer can be created as following:

```dart
// Fetch the suggested transaction params
final params = await algorand.getSuggestedTransactionParams();

// Build the transaction
final transactionA = await (PaymentTransactionBuilder()
    ..sender = accountA.address
    ..note = 'Atomic transfer from account A to account B'
    ..amount = Algo.toMicroAlgos(1.2)
    ..receiver = accountB.address
    ..suggestedParams = params)
  .build();

final transactionB = await (PaymentTransactionBuilder()
    ..sender = accountB.address
    ..note = 'Atomic transfer from account B to account A'
    ..amount = Algo.toMicroAlgos(2)
    ..receiver = accountA.address
    ..suggestedParams = params)
  .build();

// Combine the transactions and calculate the group id
AtomicTransfer.group([transactionA, transactionB]);

// Sign the transactions
final signedTxA = await transactionA.sign(accountA);
final signedTxB = await transactionB.sign(accountB);

// Send the transactions
final txId = await algorand.sendTransactions([signedTxA, signedTxB]);
```

## Asset Management

**Create a new asset**

Creating a new asset is as simple as using the ```AssetManager``` included in the Algorand SDK:

```dart
final transactionId = await algorand.assetManager.createAsset(
    account: account,
    assetName: 'FlutterCoin',
    unitName: 'Flutter',
    totalAssets: 10000,
    decimals: 2,
);
```

Or as usual, you can use the ```TransactionBuilder``` to create your asset:

```dart
// Fetch the suggested transaction params
final params = await transactionRepository.getSuggestedTransactionParams();

final transaction = await (AssetConfigTransactionBuilder()
      ..assetName = 'FlutterCoin'
      ..unitName = 'Flutter'
      ..totalAssetsToCreate = 10000
      ..decimals = 2
      ..defaultFrozen = false
      ..managerAddress = account.address
      ..reserveAddress = account.address
      ..freezeAddress = account.address
      ..clawbackAddress = account.address
      ..sender = account.address
      ..suggestedParams = params)
    .build();

// Sign the transactions
final signedTransaction = await transaction.sign(account);

// Send the transaction
final txId = await transactionRepository.sendTransaction(signedTransaction);
```

**Edit an asset**

After an asset has been created only the manager, reserve, freeze and clawback accounts can be changed.
All other parameters are locked for the life of the asset.

If any of these addresses are set to "" that address will be cleared and can never be reset for the life of the asset.
Only the manager account can make configuration changes and must authorize the transaction.

```dart
algorand.assetManager.editAsset(
    assetId: 14618993,
    account: account,
    managerAddress: account.address,
    reserveAddress: account.address,
    freezeAddress: account.address,
    clawbackAddress: account.address,
);
```

**Destroy an asset**

```dart
algorand.assetManager.destroyAsset(assetId: 14618993, account: account);
```

**Opt in to receive an asset**

Before being able to receive an asset, you should opt in
An opt-in transaction is simply an asset transfer with an amount of 0, both to and from the account opting in.
Assets can be transferred between accounts that have opted-in to receiving the asset.

```dart
algorand.assetManager.optIn(assetId: 14618993, account: account);
```

**Transfer an asset**

Transfer an asset from the account to the receiver.
Assets can be transferred between accounts that have opted-in to receiving the asset.
These are analogous to standard payment transactions but for Algorand Standard Assets.

```dart
algorand.assetManager.transfer(assetId: 14618993, account: account, receiver: receiver, amount: 1000);
```

**Freeze an asset**

Freezing or unfreezing an asset requires a transaction that is signed by the freeze account.

Upon creation of an asset, you can specify a freeze address and a defaultfrozen state.
If the defaultfrozen state is set to true the corresponding freeze address must issue unfreeze transactions,
to allow trading of the asset to and from that account.
This may be useful in situations that require holders of the asset to pass certain checks prior to ownership.

```dart
algorand.assetManager.freeze(
    assetId: 14618993,
    account: account,
    freezeTarget:
    newAccount.address,
    freeze: true,
)
```

**Revoking an asset**

Revoking an asset for an account removes a specific number of the asset from the revoke target account.
Revoking an asset from an account requires specifying an asset sender (the revoke target account) and an
asset receiver (the account to transfer the funds back to).

```dart
algorand.assetManager.revoke(
    assetId: 14618993,
    account: account,
    amount: 1000,
    revokeAddress: account.address,
  );
```

## Indexer
Algorand provides a standalone daemon algorand-indexer that reads committed blocks from the Algorand blockchain and
maintains a local database of transactions and accounts that are searchable and indexed.

The Dart SDK makes it really easy to search the ledger in a fluent api and enables application developers to perform rich and efficient queries on accounts,
transactions, assets, and so forth.

At the moment we support queries on transactions, assets and accounts.

### Transactions
Allow searching all transactions that have occurred on the blockchain.

```dart
final transactions = await algorand
  .indexer()
  .transactions()
  .whereCurrencyIsLessThan(Algo.toMicroAlgos(1000))
  .whereCurrencyIsGreaterThan(Algo.toMicroAlgos(500))
  .whereAssetId(14618993)
  .whereNotePrefix('Flutter')
  .whereTransactionType(TransactionType.PAYMENT)
  .search(limit: 5);
```

### Assets
Allow searching all assets that are created on the blockchain.

```dart
final assets = await algorand
  .indexer()
  .assets()
  .whereCurrencyIsLessThan(Algo.toMicroAlgos(1000))
  .whereCurrencyIsGreaterThan(Algo.toMicroAlgos(500))
  .whereAssetId(14618993)
  .whereUnitName('Flutter')
  .whereCreator(account.publicAddress)
  .search(limit: 5);
```
### Accounts
Allow searching all accounts that are created on the blockchain.

```dart
final accounts = await algorand
      .indexer()
      .accounts()
      .whereCurrencyIsLessThan(Algo.toMicroAlgos(1000))
      .whereCurrencyIsGreaterThan(Algo.toMicroAlgos(500))
      .whereAssetId(14618993)
      .whereAuthAddress(account.publicAddress)
      .search(limit: 5);
```
## Roadmap
* Better support for Big Integers
* Participation in consensus
* KMD
* Smart contracts
* Authorization & rekeying
* Tests

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing & Pull Requests
Feel free to send pull requests.

Please see [CONTRIBUTING](.github/CONTRIBUTING.md) for details.

## Credits

- [Tomas Verhelst](https://github.com/rootsoft)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[pub-dev-shield]: https://img.shields.io/pub/v/algorand_dart?style=for-the-badge
[pub-dev-url]: https://pub.dev/packages/algorand_dart
[effective-dart-shield]: https://img.shields.io/badge/style-effective_dart-40c4ff.svg?style=for-the-badge
[effective-dart-url]: https://github.com/tenhobi/effective_dart
[stars-shield]: https://img.shields.io/github/stars/rootsoft/algorand-dart.svg?style=for-the-badge&logo=github&colorB=deeppink&label=stars
[stars-url]: https://packagist.org/packages/rootsoft/algorand-dart
[issues-shield]: https://img.shields.io/github/issues/rootsoft/algorand-dart.svg?style=for-the-badge
[issues-url]: https://github.com/rootsoft/algorand-dart/issues
[license-shield]: https://img.shields.io/github/license/rootsoft/algorand-dart.svg?style=for-the-badge
[license-url]: https://github.com/RootSoft/algorand-dart/blob/master/LICENSE
