# dashbordの証明書

## 証明書の発行

証明書の発行にはmkcertを使います。  
mkcertのセットアップについてはこちらを参照。

証明書の発行は以下のコマンド

```
mkcert localhost
```

実行したディレクトリに以下の2つのファイルができるはずです。

- localhost-key.pem (サーバ鍵)
- localhost.pem (サーバ証明書)

これらをsecretとして登録します。

```
kubectl create secret generic kubernetes-dashboard-certs --from-file=tls.crt=./localhost.pem --from-file=tls.key=./localhost-key.pem -n kubernetes-dashboard
```

すでにdashbordを起動している場合は同名のsecretがあるので失敗します。  
その場合は先に既存のsecretを削除してください。

```
kubectl delete secret kubernetes-dashboard-certs -n kubernetes-dashboard
```

dashbordで生成した証明書が使えるようにDeploymentを修正します。

```
args:
  #- --auto-generate-certificates   <- ここをコメントアウト
  - --namespace=kubernetes-dashboard
  # Uncomment the following line to manually specify Kubernetes API server Host
  # If not specified, Dashboard will attempt to auto discover the API server and connect
  # to it. Uncomment only if the default does not work.
  # - --apiserver-host=http://my-address:port
  - --tls-cert-file=/tls.crt <- 追記
  - --tls-key-file=/tls.key <- 追記
```

外からアクセスできるようにserviceにnodePortを追加します。

```
spec:
  ports:
  - port: 443
    targetPort: 8443
    nodePort: 31111 <- 追記
```

最後のkubectl applyします

```
kubectl apply -f dashbord.yaml
```

これで https://localhost:31111 に証明書エラーなしにアクセスできるようになっているはずです
