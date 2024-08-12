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

