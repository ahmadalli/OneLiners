# OneLiners
 OneLiner commands that helped me on some scenarios


# Kubectl

## Getting Envs From All Over The Cluster

```bash
k get --all-namespaces all -o go-template='{{range .items}}{{if .spec.template}}{{$namespace := .metadata.namespace}}{{ $instance := index .spec.template.metadata.labels "app.kubernetes.io/instance"}}{{if .spec.template.spec}}{{range .spec.template.spec.containers}}{{if .env}}{{range .env}}{{$namespace}}:{{$instance}}:{{.name}}={{.value}}{{"\n"}}{{end}}{{end}}{{end}}{{end}}{{end}}{{end}}' | sort | uniq > k8s.env
```

I know this looks a little complex. Here's the multiline version of the go template:

```go
{{range .items}}
  {{if .spec.template}}
    {{$namespace := .metadata.namespace}}
    {{ $instance := index .spec.template.metadata.labels "app.kubernetes.io/instance"}}
    {{if .spec.template.spec}}
      {{range .spec.template.spec.containers}}
        {{if .env}}
          {{range .env}}
          {{$namespace}}:{{$instance}}:{{.name}}={{.value}}{{"\n"}}
          {{end}}
        {{end}}
      {{end}}
    {{end}}
  {{end}}
{{end}}
```

## Delete Evicted Pods

```bash
kubectl get pods --all-namespaces | grep Evicted | awk {'print $1" " $2'} | while read ln; do kubectl delete pod -n $ln; done
```

## Delete Terminating Pods

```bash
kubectl get pods --all-namespaces | grep Terminating | awk {'print $1" " $2'} | while read ln; do kubectl delete pod --grace-period=0 --force -n $ln; done
```

# JavaScript

## Scrool to The Buttom of Lazy Loaded Page

```js
function scrollToBottom() {
    window.scrollTo(0, document.body.scrollHeight);
}

setInterval(scrollToBottom, 100)
```
