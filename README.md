# HPA-sample
hands-on repository for Horizontal Pod Autoscaler

# 構成
Deployment(echoserver + HPA) + load test client(hey)

# ファイル
* base.yml
  * 前述の構成を含むmanifest
  * echoserverのreplicasは3を明示的に設定
* no_replicas.yml
  * base.ymlからechoserverのreplicsを削除したもの
* with_labels.yml
  * base.ymlのechoserverに追加のラベルを足したもの
  * ラベルに意味はなくDeploymentを変更するためだけのもの
* with_labels_no_replicas.yml
  * with_labels.ymlからechoserverのreplicasを削除したもの

# Case1: 適用済みのDeploymentにreplicasを明示したDeploymentをapplyする
Pod数が一時的にapplyしたDeploymentのreplicasの値になることを確認する。

1. kubectl apply -f `base.yml`
  * echoserverとload testが開始
  * replicaの上限数10になるまでPodが増えることが確認できる
  * HPAがDeploymentを見つけられない場合がある
    * オートスケールしない場合はbase.ymlを再度applyする
2. kubectl apply -f `with_labels.yml`
  * 一時的にPodが3つまで減少し、再度10まで増加することが確認できる

終わったら `kubectl delete namespace hpa-test` で掃除しておく。

# Case2: replicasを明示したDeploymentにreplicasを持たないDeploymentをapplyする
一時的にPod数が1になることを確認する。

1. kubectl apply -f `base.yml`
  * Pod数が10まで増加することを確認する
2. kubectl apply -f `no_replicas.yml`
  * Pod数が1になることを確認する
  * その後Pod数が10に戻ることを確認する
3. kubectl apply -f `with_labels_no_replicac.yml`
  * Pod数が10のままであることを確認する

終わったら `kubectl delete namespace hpa-test` で掃除しておく。

# Case3: 最初からreplicasが無いDeploymentを適用する
applyしてもPod数が変わらないことを確認する。

1. kubectl apply -f `no_replicas.yml`
  * Pod数が1で起動し、10まで増加することを確認する
2. kubectl apply -f `with_labels_no_replicas.yml`
  * Pod数が10のままであることを確認する

終わったら `kubectl delete namespace hpa-test` で掃除しておく。

# Case4: 適用済みのDeploymentを修正してからreplicas無しのDeploymentをapplyする
applyしてもPod数が変わらないことを確認する。

1. kubectl apply -f `base.yml`
  * Pod数が10まで増加することを確認する
2. kubectl -n hpa-test apply edit-last-applied deployment echoserver
  * replicasを手動で削除する
  * 削除後もPod数が10のままであることを確認する
3. kubectl apply -f `no_replicas.yml`
  * Pod数が10のままであることを確認する

終わったら `kubectl delete namespace hpa-test` で掃除しておく。

# Case5: replicasの無いDeploymentを直接修正してreplicasを指定後、replicasの無いDeploymentをapplyする
手動での編集ではreplicasが指定されているDeploymentとは判定されないことを確認する

1. kubectl apply -f `no_replicas.yml`
  * Pod数が10まで増加することを確認する
2. kubectl -n hpa-test edit deployment echoserver
  * replicasを3に修正する
  * Pod数が一時的に3になったあと、10まで増加することを確認する
3. kubectl apply -f `with_labels_no_replicas.yml`
  * Pod数が10のままであることを確認する

終わったら `kubectl delete namespace hpa-test` で掃除しておく。
