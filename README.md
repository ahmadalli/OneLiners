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

## Scroll to The End of a Lazy Loaded Page

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

## Http Request Timing

thanks to [this page](https://netbeez.net/blog/http-transaction-timing-breakdown-with-curl/). I've just added `http_code` to it

```bash
curl -o /dev/null -s <url> -w "date +%Y-%m-%dT%H:%M:%S`\ncode:          %{http_code}\nlookup:        %{time_namelookup}\nconnect:       %{time_connect}\nappconnect:    %{time_appconnect}\npretransfer:   %{time_pretransfer}\nredirect:      %{time_redirect}\nstarttransfer: %{time_starttransfer}\ntotal:         %{time_total}\n------------------------------\n"
```

## Moving LVM Volumes Between Hosts

### Requirements

- access from source host to both hosts
- stable network on the host that runs this script to the source host

you can setup ssh-agent, then ssh into the source hosts and run this script

- you must fill `target_host`, `lvm_group`, and `lvm_name` variables before running it (it's fillable and not a argument so you doable check before running it)

### The Script

run this on the source host

```bash
#!/bin/sh

target_host=""
lvm_group=""
lvm_name=""

if [[ ! -z $target_host ]] && [[ ! -z $lvm_group ]] && [[ ! -z $lvm_name ]]; then
  lvm_path="/dev/$lvm_group/$lvm_name"
  lvm_size=$(lvs $lvm_path -o LV_SIZE --noheadings --units g | xargs)
  read -p "copying $lvm_path with size of $lvm_size to $target_host. is it ok? (y/n) " -n 1 -r
  echo # (optional) move to a new line
  if [[ $REPLY =~ ^[Yy]$ ]]; then
    echo "creating volume on target"
    ssh $target_host lvcreate -n $lvm_name $lvm_group -L $lvm_size
    echo "activating volume"
    lvchange -a y $lvm_path
    echo "moving volume to the target"
    dd if=$lvm_path bs=4M | ssh $target_host dd of=$lvm_path bs=4M
  fi
else
  echo "you must fill target_host, lvm_group, and lvm_name variables"
fi

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

# OpenNebula

## RAM, CPU, HDD, and Residence Stats of VMS on CSV

sunstone doesn't include ram, cpu and hdd on the list page and I needed it for detailed host stats, so here it is. 

### Tools Required

- onevm access
- jq
  ```bash
  sudo wget https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 -O /usr/local/bin/jq
  sudo chmod +x /usr/local/bin/jq
  ```
- yq
  ```bash
  pip install yq
  ```

### The Script

```bash
#!/bin/bash

set -e

echo "downloading vm info"
for id in `onevm list | cut -d' ' -f4 | grep .`; do 
  onevm show $id --xml > $id.xml; 
done
echo "merging vm infos"
echo "<VMS>" > vms
cat *.xml >> vms
echo "</VMS>" >> vms
rm *.xml
echo "extracting stats"
cat vms | xq '[.VMS.VM[] | { id: .ID, name: .NAME, cpu: (.TEMPLATE.CPU | tonumber), memory: ((.TEMPLATE.MEMORY | tonumber) / 1024), disk: ((.TEMPLATE.DISK.SIZE | tonumber) / 1024), hostname: (.HISTORY_RECORDS.HISTORY | if type != "array" then [.] else . end | last).HOSTNAME}]' > data.json
rm vms
cat data.json | jq -r '(.[0] | keys_unsorted) as $keys | $keys, map([.[ $keys[] ]])[] | @csv' > data.csv
rm data.json
```
