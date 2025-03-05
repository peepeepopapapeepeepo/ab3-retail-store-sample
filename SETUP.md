# POC | AnyCompany

## Setup

``` bash
kubectl apply -f deploy/namespaces.yaml
kubectl apply -f deploy/assets
kubectl apply -f deploy/carts
kubectl apply -f deploy/catalog
kubectl apply -f deploy/checkout
kubectl apply -f deploy/orders
kubectl apply -f deploy/ui
```

## Cleanup

``` bash
kubectl delete ns assets
kubectl delete ns carts
kubectl delete ns catalog
kubectl delete ns checkout
kubectl delete ns orders
kubectl -n ui delete deploy ui
kubectl -n ui delete svc ui
kubectl -n ui delete cm ui
```