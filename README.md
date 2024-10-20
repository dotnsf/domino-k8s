# Domino on Kubernetes(OCP)

HCL Domino(on Docker) を Kuernetes(OCP) 上でスケール可能な形で稼働する


## HCL Domino の特徴

- データファイルを /local/notesdata 以下に格納
  - Volume 内に格納するべき
  - サーバーをスケールアウトした時に同一ファイルを参照できるようにする

## ベストプラクティスっぽいもの

- データディレクトリ用 PVC を作成
  - `$ oc apply -f domino-pvc.yaml`

- セットアップ用 Pod を作成
  - `$ oc apply -f domino-setup-deployment.yaml`
    - 起動するが RESTART する？
  - `$ oc apply -f domino-setup-service.yaml`

- ポートフォワードで動作確認＆セットアップ
  - `$ oc port-forward service/domino-setup-service 8585:8585`

- Domino Remote Setup
  - `localhost:8585`

- セットアップ終了
  - `$ oc delete -f domino-setup-service.yaml`
  - `$ oc delete -f domino-setup-deployment.yaml`

- 起動
  - `$ oc apply -f domino-deployment.yaml`
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


