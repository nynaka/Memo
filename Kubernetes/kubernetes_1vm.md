Control Plane で Pod を起動する
===

> [!NOTE]
> Kubernetes ではセキュリティ上の観点から、デフォルトでは、コントロールプレーンで Pod は起動しないようになっているそうです。  
> なので、1 台の PC で Control Plane と Worker Node を運用する場合は、このデフォルト設定を無効にする必要があります。
> ※ 1 台の PC で Kubernetes を運用する場合は minikube を使った方が手っ取り早いです。

## Control Plane ノードで Pod を起動できるようにする

コントロールプレーンとワーカーノードを異なる PC で運用すれば問題無いのですが、1 PC でコントロールプレーンとワーカーノードを運用しようとすると Pod がいつまでたっても起動しなし問題が発生します。  
下記のコマンドを実行すると解除できます。

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
kubectl taint nodes --all node-role.kubernetes.io/master-
```

※ 2 つめのコマンドは失敗すると思います。
