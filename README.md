# OneLiners
 OneLiner commands that helped me on some scenarios


# Kubectl

getting envs from all over the cluster

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
