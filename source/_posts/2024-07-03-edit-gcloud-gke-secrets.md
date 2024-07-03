---
title: 變更 Google Cloud GKE Secrets 值
date: 2024-07-03 17:04:10
tags:
---

```sh
# 取得 secret 列表
$ kubectl get secrets --all-namespaces

# base64 新資料
$ echo -n "<new-value>" | base64

# 更新 <key> 和使用 base64 值更新 <new-value>
$ kubectl patch secret <secret-name> -n <namespace> --type='json' -p='[{"op": "replace", "path": "/data/<key>", "value":"<new-value>"}]'

# 驗證資料是否更新
$ kubectl get secret <secret-name> -n <namespace> -o jsonpath='{.data.<key>}' | base64 --decode

# 重新部署
$ kubectl rollout restart deployment <deployment-name> -n <namespace>
```