# behaviour analytical
we can now trace sys calls by using tools like strace, tracee etc but still that will generate huge ampunt of data. Hence we
 need something which can trace the behaviour of the syscalls or processes. For example, in a container, why would anyone 
 need to access /etc/shadow file? this can be a risk. we need tools who can analyze this huge sys calls data.

 # Falco
 - Falco needs to see what syscalls are coming from applications so it needs to sit between application and kernel. for this it
 needs to run code in kernel space. it does this by falco kernel module. but this is very intrusive and some providers dont 
 allow to do this.
- Antoher wap is eBPF. this is safer than kernel module hence some providers allow this. it will run sysdig libraries in user
space to analyze the bahaviour by applying falco rules using its policy engine.
-  it can be installed as service unit or dameonset.

## detect threats
`journalctl -u falco -f`, check this log window to see the warning/alerts about user behaviour.
it has rules files. it will define the rule name, condition, output and priority of the rule.
````
- rule: detect shell inside a container
  desc: alert if a shell...
  condition: container.id != host and proc.name = bash
  output: Bash opened (user=%user.name container=%container.id)
  priority: WARNING
````

- container.id and proc.name etc are called sysdig filters.
- it can define macro's like `container.id!=host` as container

## config file
/etc/falco/falco.yaml.
this path can be seen from logs, or service unit.

it contains rule files
````
rules_file:
- /etc/falco/falco_rules.yaml
- /etc/falco/falco_rules.local.yaml
- /etc/falco/rules.d

priroty: debug

std_output:
 enabled: true
````

it can send output to http endpoint or slack etc.

to add custom rules, use falco_rules.local.yaml. it can update or add new rules here. dont use 
falco_rules.yaml as it can be overridden when upgraded.

To hot reload the changes to rules or any config. `kill -1 <pid>`. pid of falco process.

# Immutable vs Mutable Infrastructure

## mutable
we have a application deploye on 3 nodes. we want to upgrade the application from v1.17 to 1.19. suppose we are able to update first 2 nodes then failed at node 3 at version 1.18. There is a drift. and it is complex now. This is called in place upgrade. This type of infra is called mutable.

## immutable
same scenario above but we upgrade by adding a new node of 1.19 version and then remove node 1. we continue the same and replace all the nodes. this is called immutable infra.

## ensure immutability
- somebody can go to the container as exec and change the version so we can avoid that by add security context with `readOnlyRootFilesystem: true` but this will break the application as there wont be possible to write anything for application. to avoid that we can mount the volume for allowing write to those particualar dir.
  ````
  volumeMounts:
    - name: cache-vol
      mountPath: /var/cache/nginx
    - name: runtime-vol
      mountPath: /var/run
  volumes:
   - name: cache-vol
     emptyDir: {}
   - name: runtime-vol
     emptyDir: {}
  ````
  - Suppose we set the `previleged: true` in above. Lets try to write now.`kubectl exec -ti nginx -- apt update`. this will throw error as we have enabled the read only rootfilesystem.
    But lets try with `kubectl exec -ti nginx -- bash -c "echo '75' > /proc/sys/vm/swappiness"`. This will update the value in container but also on host too. so this is dangerous.

 In summary
 
 - always use readonly root filesystem with volume mounts as empty dir to
 - set previleged=false
 - set runAsUser=0

To ensure all of the aboev, use Pod security policy or PSA. In general stick to the POLP.


# Auditing
To log the events generated in k8s is called auditing. events are like create pods, deleting pods, secrets etc etc.. everything in cluster.

````
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
````

above is logging all the events at metadata level. to make it more specific like only deleting pods in specific namespace
````
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages: ["RequestRecieved"]
rules:
- level: RequestResponse
  namespaces: ["prod-ns"]
  verbs: ["delete"]
  resources:
    - group: " "
      resources: ["Pods"]
      resourceNames: ["pod-webapp"]
- level: Metadata
  resources:
    - group: " "
      resources: ["Secrets"]
````
above will write everything for pod deletion with name pod-webapp and metdadat for all secrets.

To enable auditing, Update the kubeapiserver with `--audit-log-path` and `--audit-policy-file`
