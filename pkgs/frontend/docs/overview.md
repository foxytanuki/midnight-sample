# Midnightブロックチェーン フロントエンド開発 技術概要

## Midnightブロックチェーンとは

Midnightは、プライバシー保護を重視したブロックチェーンです。以下の特徴があります：

* ゼロ知識証明（ZK）技術: トランザクションの詳細を隠蔽しながら、正当性を証明
* TypeScript開発: スマートコントラクトとDAppの開発にTypeScriptを使用
* UTXOモデル: Bitcoinと同様のUTXO（Unspent Transaction Output）モデルを採用
* プライベートステート: コントラクトの状態をプライベートに保持可能

## 基本的な概念

### プライベートステートとパブリックステート

Midnightのコントラクトは、2種類の状態を持ちます：

#### パブリックステート（Ledger State）

* 公開台帳に保存される状態
* 誰でも閲覧可能
* 例：カウンターの現在値、残高など

#### プライベートステート（Private State）

* ユーザー側で暗号化して保存される状態
* 他のユーザーからは見えない
* 例：ユーザーの秘密情報、個人設定など

### ゼロ知識証明（ZK Proof）

Midnightでは、トランザクションの正当性を証明するためにゼロ知識証明を使用します：

* Proof Server: ZK証明の生成を行うサーバー
* Proof Provider: フロントエンドからProof Serverに接続するためのプロバイダー

ローカル開発では、DockerでProof Serverを起動する必要があります：

```bash
docker run -p 6300:6300 midnightnetwork/proof-server -- 'midnight-proof-server --network testnet'
```

## Mesh SDK

このプロジェクトでは、Mesh SDK（`@meshsdk/midnight-setup`）を使用します。

### プロバイダーの設定

Mesh SDKでは、以下のプロバイダーを設定します：

* `privateStateProvider`: プライベートステートの保存・取得
* `zkConfigProvider`: ZK設定の取得
* `proofProvider`: Proof Serverへの接続
* `publicDataProvider`: パブリックデータ（コントラクト状態など）の取得
* `walletProvider`: ウォレット機能（トランザクションのバランス調整と証明生成）
* `midnightProvider`: Midnightネットワークへの接続（トランザクション送信）

詳細な設定方法は[README.md](../README.md)を参照してください。

### コントラクト操作

Mesh SDKでは、以下のAPIを使用してコントラクトを操作します：

* `MidnightSetupAPI.deployContract()`: コントラクトのデプロイ
* `MidnightSetupAPI.joinContract()`: 既存コントラクトへの参加
* `api.getContractState()`: コントラクト状態の取得
* `api.getLedgerState()`: レジャー状態の取得

### トランザクション処理フロー

1. コントラクトメソッドの呼び出し
2. ウォレットによるトランザクションのバランス調整（手数料の追加など）
3. Proof ServerによるZK証明の生成
4. トランザクションの送信

Mesh SDKでは、これらの処理が自動的に行われます。

## フロントエンド実装時の注意点

### 非同期処理

MidnightのAPIはすべて非同期です。適切に`async/await`を使用してください：

```typescript
const handleIncrement = async () => {
  try {
    setLoading(true);
    const result = await api.callTx.increment();
    console.log('Success:', result.public.txId);
  } catch (error) {
    console.error('Error:', error);
  } finally {
    setLoading(false);
  }
};
```

### エラーハンドリング

トランザクション送信時は、適切なエラーハンドリングを実装してください。

### ローディング状態

トランザクション処理には時間がかかります。ユーザーに適切なフィードバックを提供してください。

## 参考資料

* [Mesh公式ドキュメント](https://meshjs.dev/)
* [Midnight Network Documentation](https://docs.midnight.network/)
