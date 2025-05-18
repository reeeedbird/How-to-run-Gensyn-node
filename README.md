# 🚀 Gensynノードの起動方法（vast.ai + RTX 4090）
このドキュメントでは、vast.ai でGPUインスタンス（例: RTX 4090）をレンタルして、Gensynノードをセットアップ・起動する方法を解説します。

# ✅ 事前準備
## vast.ai のアカウント

起動済みのGPUインスタンス（例: RTX 4090）

Jupyter Notebook 経由でアクセスできること

🧰 セットアップ手順
1. インスタンスにJupyter経由で接続
vast.ai のダッシュボードから「Connect via Jupyter Notebook」をクリックし、新しいターミナルタブを開いてください。

2. 必要なパッケージのインストール
bash
コピーする
編集する
apt update && apt install -y sudo
sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip iproute2
3. 初期セットアップスクリプトの実行
bash
コピーする
編集する
curl -sSL https://raw.githubusercontent.com/zunxbt/installation/main/node.sh | bash
4. screenセッションの作成（バックグラウンドで実行するため）
bash
コピーする
編集する
screen -S gensyn
画面のデタッチ方法: Ctrl + A → D

再接続: screen -r gensyn

5. Gensyn用のリポジトリをクローンしてセットアップ
bash
コピーする
編集する
cd $HOME && \
rm -rf rl-swarm && \
git clone --branch v0.4.2 https://github.com/gensyn-ai/rl-swarm.git && \
cd rl-swarm && \
touch BACKUP.md README.md UPDATE.md backup.sh gensyn.sh && \
chmod +x gensyn.sh && \
./gensyn.sh
6. Python仮想環境のセットアップ
bash
コピーする
編集する
python3 -m venv .venv
source .venv/bin/activate
7. Gensynノードの起動
bash
コピーする
編集する
./run_rl_swarm.sh

