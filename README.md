# How-to-run-Gensyn-node-

vast.aiでRTX4090をRent
インスタンスが払い出されたらJupyter経由で接続
ファイルエクスプローラーが表示されるので新しいタブでターミナルを開く
ターミナルに下記コードを順に入力

apt update && apt install -y sudo

sudo apt update && sudo apt install -y python3 python3-venv python3-pip curl wget screen git lsof nano unzip iproute2

curl -sSL https://raw.githubusercontent.com/zunxbt/installation/main/node.sh | bash

screen -S gensyn

cd $HOME && \
rm -rf rl-swarm && \
git clone --branch v0.4.2 https://github.com/gensyn-ai/rl-swarm.git && \
cd rl-swarm && \
touch BACKUP.md README.md UPDATE.md backup.sh gensyn.sh && \
chmod +x gensyn.sh && \
./gensyn.sh


python3 -m venv .venv
source .venv/bin/activate 


./run_rl_swarm.sh
