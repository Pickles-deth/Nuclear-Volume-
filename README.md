これが出てきたんだけどなぜか真ん中だけ認識されてしまった。本来はこの白い核すべてが核ですが

月曜日 23:00
ナイス！これめちゃくちゃいい！ただ、ものによっては、写真のように白丸から大幅に外れてしまうのがあるんだけどどうすればいいか

この画像は、白い核の外側にある弱い明るさまで同じ連結領域として拾ってしまった状態です。緑が画像端まで到達しているのがそのサインです。

なので、単にしきい値を手動調整するのではなく、対策版を作りました。

この版では基本機能は変えず、次を追加しています。

    MIPから白い核の2D外形を自動検出

    その外形を「ガード領域」にする

    3D核マスクはそのガードの外へ絶対に広がらない

    それでもマスクがROIのXY端に触れた場合は、しきい値を自動的に厳しくして再試行

    体積は最終的なガード後3Dマスクから計算

左側に新しく、

核外形ガード余白 (px) = 10

があります。

まずは、

白い核の境界しきい値 = 0.82
核外形ガード余白 = 10 px

で試してください。

今回のようにまだ外へ広がる場合は、

核外形ガード余白
10 → 6 → 4

と小さくします。

逆に、本当の白い核の端まで切れてしまう場合は、

10 → 14 → 18

と大きくします。

また結果表に XY border touch も追加しました。

XY border touch = False

なら正常候補です。

XY border touch = True

なら、まだ核マスクがROI端まで漏れているので、その結果は体積としてそのまま採用しない方がいいです。

今回の画像のようなケースをかなり直接的に防ぐ修正です。理想は、緑色が白い丸の外周をほぼ沿って囲み、黒い背景部分には広がっていない状態です。

今日 9:45
これをstreamlitでリンクを知っている人が使えるようにgithubに上げたい

できます。流れは GitHubにコードを置く → Streamlit Community CloudからそのGitHubリポジトリを指定して公開 → streamlit.app のURLを共有 です。Community CloudはGitHubリポジトリから直接デプロイできます。

まずGitHubのリポジトリには、最低でもこの2つを置きます。

your-repository/
├─ app.py
└─ requirements.txt

app.py は今作ったコードを名前変更したものでOKです。

今回のコードなら requirements.txt はまずこれでよいです。

streamlit
numpy
pandas
tifffile
plotly
scipy
scikit-image
matplotlib
tensorflow
stardist
csbdeep

Streamlit Community Cloudでは、Pythonライブラリは requirements.txt で指定する必要があります。
GitHubへ上げる手順

GitHubで新しいリポジトリを作成して、

app.py
requirements.txt

をアップロードします。

リポジトリをPublicにしても構いません。Publicならコード自体は誰でも見られます。一方、コードを見せたくない場合はPrivateリポジトリでもCommunity Cloudからデプロイできますが、GitHub側でStreamlitへの追加権限が必要です。

次にStreamlit Community Cloudへ行きます。

Streamlit Community Cloud

GitHubでログインして、

Create app
↓
Yup, I have an app
↓
Repository
Branch
Main file path

を指定します。

例えば、

Repository:
あなたのGitHub名 / nuclear-volume-app

Branch:
main

Main file path:
app.py

です。

Deployすると、

https://○○○.streamlit.app

というURLが発行されます。これをLINEやメールなどで渡せば、リンクを開いた人がブラウザ上からTIFFをアップロードして解析できます。

ただし、今回のアプリについてかなり重要な注意点があります。

今のアプリは StarDist3D + TensorFlow + 3D TIFF なのでかなり重いです。Streamlit Community Cloudは公式資料上、メモリがおおよそ最大2.7GB、CPUも最大2コア程度と案内されています。上限を超えると遅くなったり、アプリが停止する可能性があります。

なので、

「GitHubに上げれば公開できる」→ YES
「今の重いStarDist3D解析がCommunity Cloud上でも確実に動く」→ まだ保証できない

という状態です。

特に以前PCでもMemoryErrorが出ていたので、Community Cloudではここを確認する必要があります。

私なら次に、今の 3D_nuclear_volume_WHITE_NUCLEUS_GUARDED.py をStreamlit Community Cloud公開用に整理して、

app.py
requirements.txt
README.md

のGitHubにそのまま3ファイル置けば動く形までこちらで作ります。

それなら、あなたはGitHubにドラッグ＆ドロップしてDeployするだけになります。

お願いします

用意しました。GitHubにそのまま置ける3ファイル一式です。

GitHubのリポジトリ直下を、

app.py
requirements.txt
README.md

にしてください。Streamlit Community Cloudは、エントリポイントと同じ場所かリポジトリ直下の requirements.txt を読み込んで依存関係をインストールします。

その後、Streamlit Community Cloudで、

Repository     あなたのGitHubリポジトリ
Branch         main
Main file path app.py

を指定してDeployすればOKです。PythonのバージョンはDeploy時の Advanced settings から指定できます。

一点だけ重要で、このアプリは TensorFlow + StarDist3D + 3D TIFF を使うため、公開自体はできても、Community CloudのCPU/RAM制限で大きな画像の解析が重くなる可能性があります。Community Cloudにはリソース上限があります。

次はGitHubでどこを押して3ファイルをアップロードするかを、画面順に案内できます。


ライブラリ/README.md
3D Nuclear Volume Analyzer

Streamlit app for measuring 3D nuclear volume from DAPI Z-stack TIFF images.
Features

    Upload multiple 2D TIFF slices as one Z-stack

    Detect candidate nuclear ROIs

    Use StarDist3D as a seed detector

    Expand the seed to the whole bright DAPI nucleus

    Guard against mask leakage outside the visible nuclear outline

    Calculate nuclear volume from voxel count

    Inspect segmentation overlays

    Optional 3D mesh display

    Export CSV

Volume formula

Volume (µm³) = voxel count × Pixel X × Pixel Y × Z spacing

The final volume calculation does not resize or downsample the image.
Repository structure

.
├── app.py
├── requirements.txt
└── README.md

Run locally

pip install -r requirements.txt
streamlit run app.py

Deploy on Streamlit Community Cloud

    Create a GitHub repository.

    Upload app.py, requirements.txt, and README.md to the repository root.

    Open Streamlit Community Cloud.

    Click Create app.

    Select the GitHub repository and branch.

    Set Main file path to app.py.

    In Advanced settings, choose the Python version you want to use.

    Click Deploy.

Streamlit will provide a *.streamlit.app URL that you can share.
Important

This app uses TensorFlow + StarDist3D and 3D TIFF data, so it can require substantial RAM and CPU.
Large analyses may exceed Streamlit Community Cloud resource limits even when deployment succeeds.

Always visually confirm that the colored segmentation mask matches the full white DAPI nucleus before using the measured volume.
