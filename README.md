# 🚀 Gensynノードの起動方法（vast.ai + RTX 4090）
このドキュメントでは、vast.ai でGPUインスタンス（例: RTX 4090）をレンタルして、Gensynノードをセットアップ・起動する方法を解説します。

## ✅ 事前準備
・vast.ai のアカウント  
・起動済みのGPUインスタンス（例: RTX 4090）  
・Jupyter Notebook 経由でアクセスできること  

## 🧰 セットアップ手順
## 1. インスタンスにJupyter経由で接続
vast.ai のダッシュボードから「Connect via Jupyter Notebook」をクリックし、新しいターミナルタブを開く

## 2. 必要なパッケージのインストール
ターミナルに下記入力
```
apt update && apt install -y sudo
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip iproute2
```

## 3. 初期セットアップスクリプトの実行
ターミナルに下記入力
```
curl -sSL https://raw.githubusercontent.com/zunxbt/installation/main/node.sh | bash
```

## 4. screenセッションの作成（バックグラウンドで実行するため）
ターミナルに下記入力
```
screen -S gensyn
```
※画面のデタッチ方法: Ctrl + A → D

## 5. Gensynのリポジトリをクローンしてセットアップ
ターミナルに下記入力
```
cd $HOME && \
rm -rf rl-swarm && \
git clone --branch v0.4.2 https://github.com/gensyn-ai/rl-swarm.git && \
cd rl-swarm && \
touch BACKUP.md README.md UPDATE.md backup.sh gensyn.sh && \
chmod +x gensyn.sh && \
./gensyn.sh
```
※既存のswarm.pemを持っている場合は、このタイミングでswarm.pemをrl-swarm配下に格納しておく
Jupyterのファイルエクスプローラーでローカルからのuploadが可能
※swarm.pemは初回発行時に紐づいたメールアドレスとペアになっていて変更不可

## 6.ngrokのセットアップ
左上のFile→Newから新しいターミナルを開いて下記コードを入力
```
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
sudo mv ngrok /usr/local/bin
```
```
ngrok config add-authtoken <ngrokのAuthtoken>
```
※Authtokenは事前にngrokに登録して入手しておきましょう
```
ngrok http 3000
```

## 7. Python仮想環境のセットアップ
元のターミナルに戻り下記入力
```
python3 -m venv .venv
source .venv/bin/activate
```

## 8. バグ対応①（リダイレクトできない問題）
ターミナルに下記入力
```
nano modal-login/app/page.tsx
```
下記内容で上書き保存
```
"use client";
import {
  useAuthModal,
  useLogout,
  useSigner,
  useSignerStatus,
  useUser,
} from "@account-kit/react";
import { useEffect, useState } from "react";

export default function Home() {
  const user = useUser();
  const { openAuthModal } = useAuthModal();
  const signerStatus = useSignerStatus();
  const { logout } = useLogout();
  const signer = useSigner();

  const [createdApiKey, setCreatedApiKey] = useState(false);

  useEffect(() => {
    // User logged out, so reset the state.
    if (!user && createdApiKey) {
      setCreatedApiKey(false);
    }

    // Waiting for user to be logged in.
    if (!user || !signer || !signerStatus.isConnected || createdApiKey) {
      return;
    }

    const submitStamp = async () => {
      const whoamiStamp = await signer.inner.stampWhoami();
      const resp = await fetch("/api/get-api-key", {
        method: "POST",
        body: JSON.stringify({ whoamiStamp }),
      });
      return (await resp.json()) as { publicKey: string };
    };

    const createApiKey = async (publicKey: string) => {
      await signer.inner.experimental_createApiKey({
        name: `server-signer-${new Date().getTime()}`,
        publicKey,
        expirationSec: 60 * 60 * 24 * 62, // 62 days
      });
    };

    const handleAll = async () => {
      try {
        const { publicKey } = await submitStamp();
        await createApiKey(publicKey);
        await fetch("/api/set-api-key-activated", {
          method: "POST",
          body: JSON.stringify({ orgId: user.orgId, apiKey: publicKey }),
        });
        setCreatedApiKey(true);
      } catch (err) {
        console.error(err);
        alert("Something went wrong. Please check the console for details.");
      }
    };

    handleAll();
  }, [createdApiKey, signer, signerStatus.isConnected, user]);

  // Show alert if crypto.subtle isn't available.
  useEffect(() => {
    if (typeof window === "undefined") return;

    try {
      if (typeof window.crypto.subtle !== "object") {
        throw new Error("window.crypto.subtle is not available");
      }
    } catch (err) {
      alert(
        "Crypto API is not available in this browser. Please use HTTPS or localhost."
      );
    }
  }, []);

  useEffect(() => {
    if (!user && !signerStatus.isInitializing) {
      openAuthModal();
    }
  }, [user, signerStatus.isInitializing]);

  return (
    <main className="flex min-h-screen flex-col items-center gap-4 justify-center text-center">
      {signerStatus.isInitializing || (user && !createdApiKey) ? (
        <>Loading...</>
      ) : user ? (
        <div className="card">
          <div className="flex flex-col gap-2 p-2">
            <p className="text-xl font-bold">
              YOU ARE SUCCESSFULLY LOGGED IN TO THE GENSYN TESTNET
            </p>
            <button className="btn btn-primary mt-6" onClick={() => logout()}>
              Log out
            </button>
          </div>
        </div>
      ) : (
        <div className="card">
          <p className="text-xl font-bold">LOGIN TO THE GENSYN TESTNET</p>
          <div className="flex flex-col gap-2 p-2">
            <button className="btn btn-primary mt-6" onClick={openAuthModal}>
              Login
            </button>
          </div>
        </div>
      )}
    </main>
  );
}
```

## 9. バグ対応②（failed to connect to bootstrap peers）
ターミナルに下記入力
```
nano hivemind_exp/runner/gensyn/testnet_grpo_runner.py
```
下記内容で上書き保存
ターミナルに下記入力
※/ip4/xxxx.xxx.xxx.xxxの部分は各自異なるので要確認
```
import logging
from dataclasses import dataclass
from functools import partial
from typing import Callable, Tuple

from datasets import Dataset
from trl import GRPOConfig, ModelConfig

from hivemind_exp.chain_utils import (
    SwarmCoordinator,
)
from hivemind_exp.runner.grpo_runner import GRPOArguments, GRPORunner
from hivemind_exp.trainer.gensyn.testnet_grpo_trainer import TestnetGRPOTrainer

logger = logging.getLogger(__name__)


@dataclass
class TestnetGRPOArguments:
    # Mutually exclusive.
    wallet_private_key: str | None = None  # EOA wallet private key.
    modal_org_id: str | None = None  # Modal organization ID.

    # Swarm coordinator contract address
    contract_address: str = ""


class TestnetGRPORunner(GRPORunner):
    def get_initial_peers(self) -> list[str]:
        # Skip chain lookup if no peers provided
        if not getattr(self, 'force_chain_lookup', True):
            return []

        # Original chain lookup
        peers = self.coordinator.get_bootnodes()
        logger.info(f"Bootnodes from chain: {peers}")

        # Filter out dead peers (optional)
        alive_peers = [p for p in peers if not p.startswith('/ip4/xxxx.xxx.xxx.xxx')]
        return alive_peers if alive_peers else []

    def __init__(self, coordinator: SwarmCoordinator) -> None:
        self.coordinator = coordinator

    def register_peer(self, peer_id):
        logger.info(f"Registering self with peer ID: {peer_id}")
        self.coordinator.register_peer(peer_id)

    def setup_dht(self, grpo_args):
        dht = super().setup_dht(grpo_args)
        peer_id = str(dht.peer_id)
        self.register_peer(peer_id)
        return dht

    def run(
        self,
        model_args: ModelConfig,
        grpo_args: GRPOArguments,
        training_args: GRPOConfig,
        initial_datasets_fn: Callable[[], Tuple[Dataset, Dataset]],
    ):
        initial_peers = grpo_args.initial_peers
        if not initial_peers:
            initial_peers = self.get_initial_peers()
            logger.info(f"Retrieved initial peers from chain: {initial_peers}")
        elif initial_peers == ["BOOT"]:
            initial_peers = []
            logger.info("Proceeding as bootnode!")

        grpo_args.initial_peers = initial_peers
        super().run(
            model_args,
            grpo_args,
            training_args,
            initial_datasets_fn,
            partial(TestnetGRPOTrainer, coordinator=self.coordinator),
        )
```

## 10. Gensynノードの起動
ターミナルに下記入力
```
./run_rl_swarm.sh
```

## 11. パラメータ指定
テストネットへの接続 →y
math / math hard →b
parameter →0.5
※1.5だとエラーが出るけど0.5だとうまくいくとかあるので使っているインスタンスに依存するので要確認

## 13. 認証ページへのアクセス（リダイレクト）
localhost:3000へのアクセスを求められたら、ngrok経由でlocalhost:3000に接続
※手順6で開いたターミナルに記載のURLよりアクセス

## 14. hugging face連携
Hugging faceのAPIを使うか聞かれるので使う場合はyを入力し、自身のAPIを入力

以上

