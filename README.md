# Domino on Kubernetes(OCP)

HCL Domino(on Docker) を Kuernetes(OCP) 上でスケール可能な形で稼働する


## HCL Domino の特徴

- データファイルを /local/notesdata 以下に格納
  - Volume 内に格納するべき
  - サーバーをスケールアウトした時に同一ファイルを参照できるようにする

## ベストプラクティスっぽいもの

- docker で初期セットアップする
  - `$ docker run --rm --name mydominosetup -v /home/user01/notesdata:/local/notesdata --hostname mydomino.localdomain.com -p 8585:8585 -p 1352:1352 domino-docker:V1101_03212020prod --setup`

  - Domino Remote Setup
    - `mydomino.localdomain.com:8585`

  - `server.id`, `cert.id`, `admin.id` が手元にある状態にする
  - docker ホストの `/home/user01/notesdata` にノーツデータがある状態になっているとする

- 必要があれば、初期セットアップ後の利用（DB追加など）
  - `$ docker run -d --name mydomino -v /home/user01/notesdata:/local/notesdata --hostname mydomino.localdomain.com -p 8000:8000 -p 1352:1352 domino-docker:V1101_03212020prod`
  - HTTP ポートを 80 -> 8000 にして再起動をかけておく

- ここまでで docker で Domino 環境ができている状態になっている
- ここからコンテナクラスタ用のデータ移行

- データディレクトリ用 PVC を作成
  - `$ oc apply -f domino-pvc.yaml`

- ノーツデータ移行用テンポラリ Pod を起動して、PVC にデータを同期する
  - `$ oc apply -f dummy-nginx-pod.yaml`
    - `/local/notesdata` に上記 PVC をマウント
  - `$ oc rsync /home/user01/notesdata dummy-nginx-pod:/local`
    - 最後は `/local/notesdata` ではなく `/local` のみ
      - そうしないと `/local/notesdata/notesdata` フォルダができてしまう
    - `*.lck` ファイルは消しておく

- ここまででコンテナクラスタ用のデータが PVC 内（`/local/notesdata`）に移行出来た状態になっている
- ここから Domino サーバー稼働

- 起動
  - `$ oc apply -f domino-deployment.yaml`
    - **CrashLoopBackOff**
  - `$ oc apply -f domino-service.yaml`
  - `$ oc expose service domino-service`
  - HTTP を8000 番にして再起動


## 動作確認

- domino Pod をスケールする
  - domino Deployment から Pod 数を２以上に変更しても正しく動くことを確認する


## クリーンアップ

- 全リソース削除
  - `$ oc delete -f .`


## Reference


