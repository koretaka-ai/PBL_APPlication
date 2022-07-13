# PBL memo

# 翻訳アプリに使うモデルの作成
- small_parallel_enjaをjaparacrawl事前学習済みモデル（ベースモデル）でfinetuiningする
- python 環境 3.8.10
## git clone
~~~
git clone https://github.com/koretaka-ai/PBL_translation.git
cd PBL_translation
# branchの切り替え
git switch application
# translate ディレクトリに移動
cd translate
~~~
## Anacondaが入っていなかったら導入
- 詳しく説明しているサイト：https://hana-shin.hatenablog.com/entry/2022/02/12/203642
~~~
wget https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh
bash Anaconda3-2021.05-Linux-x86_64.sh
~~~
## 仮想環境構築
~~~
conda create -n PBL_t python=3.8.10 -y
conda activate PBL_t
~~~
## 必要なライブラリのインストール
- fairseq install 
~~~
git clone https://github.com/pytorch/fairseq
cd fairseq
pip install --editable ./
cd ..
~~~
- sentencepiece install
~~~
git clone https://github.com/google/sentencepiece.git 
cd sentencepiece
mkdir build
cd build
cmake ..
make -j $(nproc)
sudo make install
sudo ldconfig -v
cd ../..
~~~
- vcpkg install
~~~
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg integrate install
./vcpkg install sentencepiece
cd ..
~~~
- mecab install 
~~~
git clone https://github.com/taku910/mecab.git
cd mecab/mecab
./configure --enable-utf8-only
make & make check
make install
cd ../..
~~~
- mecab dict install https://drive.google.com/uc?export=download&id=0B4y35FiV1wh7MWVlSDBCSXZMTXM
~~~
tar zxfv mecab-ipadic-2.7.0-20070801.tar.gz
rm mecab-ipadic-2.7.0-20070801.tar.gz
cd mecab-ipadic-2.7.0-20070801
./configure --with-charset=utf8
make
make install
cd ..
~~~
## データの準備
- データの前処理 small_parallel_enjaのdetokenizeとjparacrawl pre-trained-modelを使うためにdownloadしたspm modelでtokenizeする
~~~
pushd scripts/preprocess
bash preprocess.sh
popd
~~~
## モデルの訓練
- fairseqの前処理と訓練
- GPUを使う際には `nvidia-smi` を使用してGPUがあいているか確認
~~~ 
pushd scripts/train
bash fairseq_preprocess.sh
bash train.sh -n ${実験名} -g ${GPUのID} -s ${SEED値}
popd
~~~
## モデルの評価
- モデルの評価にはBLUEを使う
~~~
pushd scripts/eval
bash eval-blue.sh -n ${実験名} -g ${GPUのID}
popd
~~~
## 仮想環境から抜ける
~~~
conda deactivate
cd ..
~~~
# Nginx, uWSGI webサーバを使用してFlaskアプリケーションを導入する
- python 環境 3.8.10
## 仮想環境の構築
- 仮想環境はvirtualenvで構築する
~~~
# app_venv ディレクトリに移動
cd app_venv
virtualenv app_env
source app_env/bin/activate
~~~
## 必要なライブラリのインストール
- uwsgi install
~~~
pip install uwsgi
~~~
- flask install
~~~
pip install Flask
~~~
- fairseq install 
~~~
git clone https://github.com/pytorch/fairseq
cd fairseq
pip install --editable ./
cd ..
~~~
- sentencepiece install
~~~
git clone https://github.com/google/sentencepiece.git 
cd sentencepiece
mkdir build
cd build
cmake ..
make -j $(nproc)
sudo make install
sudo ldconfig -v
cd ../..
~~~
- vcpkg install
~~~
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh
./vcpkg integrate install
./vcpkg install sentencepiece
cd ..
~~~
## アプリの起動
- 127.0.0.1:5000 に翻訳アプリが表示される
~~~
python app.py
~~~
- app.iniのhome, pythonpath, chdir, logtoのpathは各自のpathに書き換えてください
## nginxのインストールとuWSGIの設定 
- こちらのサイトでnginxとuWSGIの設定に関して詳しく説明されている 
- https://serip39.hatenablog.com/entry/2020/07/06/070000
