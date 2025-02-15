kubectl get po -A|grep vwac|grep -v tcp|awk '{print $2}'|xargs -I {} kubectl get po {} -o jsonpath="{range .spec.containers[*]}{.env} {'\n'}{end}" -n vwac

kubectl get po -A|grep vwac|grep -v tcp|awk '{print $2}'|xargs -I {} kubectl get po {} -o json -n vwac|jq -r '.spec.containers[].env'
按照创建时间排序
kubectl get cm -n podsmanager --sort-by=.metadata.creationTimestamp

https://kubernetes.io/zh-cn/docs/reference/kubectl/jsonpath/
