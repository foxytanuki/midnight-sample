# Frontend

## 概要

このプロジェクトは、Midnightブロックチェーンでのフロントエンド実装方法を確認するためのサンプルプロジェクトです。

[Mesh SDK](https://meshjs.dev/)の`@meshsdk/midnight-setup`パッケージを使用してMidnightブロックチェーンと連携します。

## 前提条件

* Lace Beta Wallet for Midnight Network: ブラウザ拡張機能としてインストールが必要
* Proof Server: ローカル開発ではDockerで起動が必要

## セットアップ

### パッケージのインストール

```bash
npm install @meshsdk/midnight-setup

npm install \
  @midnight-ntwrk/dapp-connector-api@3.0.0 \
  @midnight-ntwrk/midnight-js-fetch-zk-config-provider@2.0.2 \
  @midnight-ntwrk/midnight-js-http-client-proof-provider@2.0.2 \
  @midnight-ntwrk/midnight-js-indexer-public-data-provider@2.0.2 \
  @midnight-ntwrk/midnight-js-level-private-state-provider@2.0.2 \
  @midnight-ntwrk/midnight-js-network-id@2.0.2
```

### Proof Serverの起動

```bash
docker run -p 6300:6300 midnightnetwork/proof-server -- 'midnight-proof-server --network testnet'
```

## 基本的な使い方

### プロバイダーの設定

```typescript
import { FetchZkConfigProvider } from "@midnight-ntwrk/midnight-js-fetch-zk-config-provider";
import { httpClientProofProvider } from "@midnight-ntwrk/midnight-js-http-client-proof-provider";
import { indexerPublicDataProvider } from "@midnight-ntwrk/midnight-js-indexer-public-data-provider";
import { levelPrivateStateProvider } from "@midnight-ntwrk/midnight-js-level-private-state-provider";
import type { MidnightSetupContractProviders } from "@meshsdk/midnight-setup";

export async function setupProviders(): Promise<MidnightSetupContractProviders> {
  const wallet = window.midnight?.mnLace;
  if (!wallet) {
    throw new Error("Please install Lace Beta Wallet for Midnight Network");
  }

  const walletAPI = await wallet.enable();
  const walletState = await walletAPI.state();
  const uris = await wallet.serviceUriConfig();

  return {
    privateStateProvider: levelPrivateStateProvider({
      privateStateStoreName: "my-dapp-state",
    }),
    zkConfigProvider: new FetchZkConfigProvider(
      window.location.origin,
      fetch.bind(window),
    ),
    proofProvider: httpClientProofProvider(uris.proverServerUri),
    publicDataProvider: indexerPublicDataProvider(
      uris.indexerUri,
      uris.indexerWsUri,
    ),
    walletProvider: {
      coinPublicKey: walletState.coinPublicKey,
      encryptionPublicKey: walletState.encryptionPublicKey,
      balanceTx: (tx, newCoins) => {
        return walletAPI.balanceAndProveTransaction(tx, newCoins);
      },
    },
    midnightProvider: {
      submitTx: (tx) => {
        return walletAPI.submitTransaction(tx);
      },
    },
  };
}
```

### コントラクトのデプロイ

```typescript
import { MidnightSetupAPI } from "@meshsdk/midnight-setup";
import { setupProviders } from "./providers";

async function deployContract() {
  const providers = await setupProviders();
  const contractInstance = new MyContract({});

  const api = await MidnightSetupAPI.deployContract(
    providers,
    contractInstance,
  );

  console.log("Contract deployed at:", api.deployedContractAddress);
  return api;
}
```

### 既存コントラクトへの参加

```typescript
async function joinContract(contractAddress: string) {
  const providers = await setupProviders();
  const contractInstance = new MyContract({});

  const api = await MidnightSetupAPI.joinContract(
    providers,
    contractInstance,
    contractAddress,
  );

  return api;
}
```

### コントラクト状態の取得

```typescript
const contractState = await api.getContractState();
const ledgerState = await api.getLedgerState();
```

## 技術スタック

* TypeScript
* React（またはNext.js）
* Vite（またはNext.jsのビルドシステム）

## 参考資料

* [Mesh公式ドキュメント](https://meshjs.dev/)
* [Midnight Network Documentation](https://docs.midnight.network/)
