# OneLiners
 OneLiner commands that helped me on some scenarios


# Kubectl

## Getting Envs From All Over The Cluster

```bash
k get --all-namespaces all -o go-template='{{range .items}}{{if .spec.template}}{{$namespace := .metadata.namespace}}{{$instance := index .spec.template.metadata.labels "app.kubernetes.io/instance"}}{{if .spec.template.spec}}{{range .spec.template.spec.containers}}{{if .env}}{{range .env}}{{$namespace}}:{{$instance}}:{{.name}}={{.value}}{{"\n"}}{{end}}{{end}}{{end}}{{end}}{{end}}{{end}}' | sort | uniq > k8s.env
```

I know this looks a little complex. Here's the multiline version of the go template:

```go
{{range .items}}
  {{if .spec.template}}
    {{$namespace := .metadata.namespace}}
    {{$instance := index .spec.template.metadata.labels "app.kubernetes.io/instance"}}
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

# Nuget

## OneLiner Multiple Package Installer

you need to add this function to your `Profile.ps1` ([more info](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles))

```powershell
function nugetip ([Parameter(ValueFromRemainingArguments=$true)][string[]]$packages) {
    Foreach ($package in $packages) {
        install-package $package
    }
}
```

# Linux

## Print All Authorized Keys for All Users

```bash
cat /etc/passwd | grep -Ev "^#" | cut -d: -f6 | xargs -I {} sh -c 'file={}/.ssh/authorized_keys; if [ -f $file ]; then echo -n {} && echo ":" && cat $file | grep . && echo; fi'
```

### OSX

on osx, `/etc/passwd` is incomplete. you need to use `dscacheutil -q user | grep dir | cut -d" " -f2` for getting home directory of users instead. so on osx the command will be this:


```bash
dscacheutil -q user | grep dir | cut -d" " -f2 | xargs -I {} sh -c 'file={}/.ssh/authorized_keys; if [ -f $file ]; then printf {} && echo ":" && cat $file | grep . && echo; fi'
```

# Ansible

## Download Docker Compose

add it to `tasks` section of your ansible playbook or to the tasks of your role

```ansible
- name: install docker-compose
  get_url:
    dest: /usr/local/bin/docker-compose
    url: https://github.com/docker/compose/releases/latest/download/docker-compose-{{ansible_system}}-{{ansible_userspace_architecture}}
    mode: 755
```
