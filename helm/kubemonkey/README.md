# Kube-Monkey Helm Chart

[Kube-Monkey](https://github.com/asobti/kube-monkey) periodically kills pods in your Kubernetes cluster, that are opt-in based on their own rules.

## Add repository

```bash
helm repo add kubemonkey https://asobti.github.io/kube-monkey/charts/repo
helm repo update
```

## Installing the Chart

To install the chart with the release name `my-release`:

With Helm v3

```bash
helm install my-release kubemonkey/kube-monkey --version 1.4.0
```

With Helm v2

```bash
helm install --name my-release kubemonkey/kube-monkey --version 1.4.0
```

The command deploys kube-monkey on the Kubernetes cluster in the default configuration. The [configurations](#Configurations) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```console
$ helm delete my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Customising Configurations

By default `Kube-Monkey` runs in dry-run mode so it doesn't actually kill anything.
If you're confident you want to use it in real run `helm` with:

```console
$ helm install --name my-release kubemonkey --set config.dryRun=false
```

By default `Kube-Monkey` runs in without any white listed namespace assigned so it doesn't actually kill anything.
If you're confident you want to enable it in real, run `helm` with:

```console
$ helm install --name my-release kubemonkey \
               --set config.dryRun=false \
               --set config.whitelistedNamespaces="{namespace1,namespace2,namespace3}"
```

**Note: replace namespace with your real namespaces**

If you want to see how kube-monkey kills pods immediately in debug mode.

```console
$ helm install --name my-release kubemonkey \
               --set config.dryRun=false \
               --set config.whitelistedNamespaces="{namespace1,namespace2,namespace3}"
               --set config.debug.enabled=true \
               --set config.debug.schedule_immediate_kill=true
```
If you want to change the time kube-monkey wakes up and start and end killing pods.

```console
$ helm install --name my-release kubemonkey \
               --set config.dryRun=false \
               --set config.whitelistedNamespaces="{namespace1,namespace2,namespace3}"
               --set config.runHour=10 \
               --set config.startHour=11 \
               --set config.endHour=17 
```
If you want to enable attacks notifications.

```console
$ helm install --name my-release kubemonkey \
               --set config.dryRun=false \
               --set config.whitelistedNamespaces="namespace1\"\,\"namespace2\"\,\"namespace3" \
               --set config.notifications.enabled=true \
               --set config.notifications.endpoint=http://localhost:8080/path \
               --set config.notifications.message="{\"foo\":\"bar\"}" \
               --set config.notifications.headers="Content-Type:application/json\"\,\"client-id:kubemonkey"
```
If you want validate intended values passed in to configmap .

```console
$ helm get manifest my-release
```
## Configurations

| Parameter                              | Description                                                                             | Default                          |
|----------------------------------------|-----------------------------------------------------------------------------------------|----------------------------------|
| `image.repository`                     | docker image repo                                                                       | ayushsobti/kube-monkey           |
| `image.tag`                            | docker image tag                                                                        | v0.4.1                           |
| `replicaCount`                         | number of replicas to run                                                               | 1                                |
| `rbac.enabled`                         | rbac enabled or not                                                                     | true                             |
| `image.pullPolicy`                     | image pull logic                                                                        | IfNotPresent                     |
| `config.dryRun`                        | will not kill pods, only logs behaviour                                                 | true                             |
| `config.runHour`                       | schedule start time in 24hr format                                                      | 8                                |
| `config.startHour`                     | pod killing start time  in 24hr format                                                  | 10                               |
| `config.endHour`                       | pod killing stop time  in 24hr format                                                   | 16                               |
| `config.whitelistedNamespaces`         | pods in this namespace that opt-in will be killed                                       |                                  |
| `config.blacklistedNamespaces`         | pods in this namespace will not be killed                                               | kube-system                      |
| `config.timeZone`                      | time zone in DZ format                                                                  | America/New_York                 |
| `config.debug.enabled`                 | debug mode,need to be enabled to see debuging behaviour                                 | false                            |
| `config.debug.schedule_immediate_kill` | immediate pod kill matching other rules apart from time                                 | false                            |
| `config.notifications.enabled`         | enables reporting of attacks to an HTTP endpoint                                        | false                            |
| `config.notifications.proxy`           | notifications proxy URL                                                                 |                                  |
| `config.notifications.attacks`         | HTTP collector in the form (endpoint,message,headers) where attacks will be reported to |                                  |
| `args.logLevel`                        | go log level                                                                            | 5                                |
| `args.logDir`                          | log directory                                                                           | /var/log/kube-monkey             |

after all you can simply edit values.yaml with your preferred configs and run as below

```console
$ helm install --name my-release kubemonkey --namespace=kube-monkey
```
example of a modified values.yaml (only important parts are displayed)

```yaml
...
replicaCount: 1
rbac:
  enabled: true
image:
  repository: ayushsobti/kube-monkey
  tag: v0.4.1
  pullPolicy: IfNotPresent
config:
  dryRun: false
  runHour: 8
  startHour: 10
  endHour: 16
  blacklistedNamespaces: [ "kube-system" ]
  whitelistedNamespaces: [ "namespace1", "namespace2" ]
  timeZone: America/New_York
args:
  logLevel: 5
  logDir: /var/log/kube-monkey
...
```
