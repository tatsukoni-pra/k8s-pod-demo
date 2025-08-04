## Annotations更新時の挙動

- kustomization.yaml の `commonAnnotations` 設定値を変更すると、Podが再作成される
- これは、Deploymentの仕様で、Podテンプレートのメタデータやspecが変更されると、新しいReplicaSetが作成され、Podがローリングアップデートされるため。

## リソース反映方法

```
$ /Users/konishitatsuhiro/Desktop/git/k8s-pod-demo/k8s

# custom-app/ を反映したい場合
$ kubectl apply -k custom-app
```

## リソース情報取得方法

```
$ /Users/konishitatsuhiro/Desktop/git/k8s-pod-demo/k8s

# custom-app/ を取得したい場合
$ kubectl get -k custom-app
```

## Annotations値更新方法

Kustomization経由で作成された、DeploymentsのAnnotationsを更新する方法。<br>
この方法であれば、DeploymentsのAnnotationsのみ更新されるので、Podは再起動されない。<br>

```
# 付与
$ kubectl get -k custom-app | grep deployment | awk '{print $1}' | xargs -I {} kubectl annotate {} custom-app/active=true -n test-namespace
$ kubectl get -k other-app | grep deployment | awk '{print $1}' | xargs -I {} kubectl annotate {} other-app/active=true -n test-namespace

# 更新
$ kubectl get -k custom-app | grep deployment | awk '{print $1}' | xargs -I {} kubectl annotate {} custom-app/active=false -n test-namespace --overwrite
$ kubectl get -k other-app | grep deployment | awk '{print $1}' | xargs -I {} kubectl annotate {} other-app/active=false -n test-namespace --overwrite
```

## `*/active=true` のAnnotationが設定されているか否かの判定方法

```
$ count=$(kubectl get -k custom-app -o json | jq '[.items[] | select(.kind == "Deployment") | select(.metadata.annotations | to_entries[] | select(.key | endswith("/active")) | .value == "true")] | length')
$ [ $count -gt 0 ] && echo "EXISTS ($count deployments found)" || echo "NOT EXISTS"
```
