# Run Observer on a Kubernetes pod deployment

This is an adaptation for Kubernetes of the amazing [dominicletz](https://github.com/dominicletz) script for SSH, [remote_observe](https://github.com/dominicletz/remote_observe).

`kubernetes_observer` runs an Observer on a remote Kubernetes pod deployment, using `kubectl`. The script automates port discovery and forwarding.

You only have to ensure that in your mix.exs you've added the runtime_tools in your deployment:

```elixir
  # Run "mix help compile.app" to learn about applications.
  def application do
    [
      mod: {My.Application, []},
      extra_applications: [:runtime_tools]
    ]
  end
```

## SYNOPSIS

```
Usage: kubernetes_observer (-options) <target> (<node_name>)

  <target>    target pod or type/name.
  <node_name> is only required if there is more than one node
              running on the remote machine.

Options:

  -e                Start an erlang shell instead of an elixir shell
  -l                Use long names instead of default short names
  -k <cookie>       Set the cookie value
  -h <cookie_home>  Looks for the cookie in <cookie_home>/.erlang.cookie
                    on the remote node
  -p <cookie_path>  Looks for the cookie in <cookie_path> on the remote
                    node
  -o <options>      Set the kubectl options
  -c <container>    Set the Kubernetes cluster namespace

Example:
  kubernetes_observer -k secret my-k8s-pod
  kubernetes_observer -p /opt/app/releases/COOKIE my-k8s-deployment
```

### Optional Environment Variables

- `ERL_EPMD_PORT` - Define an alternative EPMD port

## Installation

Mark the script `kubernetes_observer` as executable and move it somewhere in your \$PATH for convinience:

### Linux

```
wget https://raw.githubusercontent.com/danielpilon/kubernetes_observer/master/kubernetes_observer
chmod +x ./kubernetes_observer
sudo mv ./kubernetes_observer /usr/local/bin
```

### MacOS (with curl)

```
curl https://raw.githubusercontent.com/danielpilon/kubernetes_observer/master/kubernetes_observer -o kubernetes_observer
chmod +x ./kubernetes_observer
sudo mv ./kubernetes_observer /usr/local/bin
```

## Notes

- [Releases with Runtime tools](#releases-with-runtime-tools)
- [kubectl](#kubectl)
- [Compatible VM](#compatible-vm)
- [Access to the Cookie](#access-to-the-cookie)
- [Short and long names](#short-and-long-names)
- [EPMD Port](#epmd-port)


### Releases with Runtime tools
In order to run the observer tool locally, your release needs to include the runtime tools. You can [read more about this here](https://tkowal.wordpress.com/2016/04/23/observer-in-erlangelixir-release/) and about [enabling distribution in releases here](https://elixirforum.com/t/remote-observer-connection-issues/26315/3)

### kubectl
To interact with a Kubernetes cluster, `kubernetes_observer` uses [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/). It's required to have it preinstalled and configured in order to use the script. 

Basically it uses the [port-foward](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) command to establish a connection between the local and the remote machine. 

`kubernetes_observer` accepts via the `-o` flag the same level of options as the `port-foward` command does. 

If multiple containers are running at the same pod, the name of the desired container can be specified via the `-c` option.

### Compatible VM

You need to have a local installed Elixir VM that is compatible with the remote Elixir VM that you have

### Access to the Cookie

To connect to a remote VM `kubernetes_observer` tries to determine the secret cookie. It assumes to be in the home directory of the pod.

1. If you are running a release the cookie can be in the releases/COOKIE subdirectory of your app. In that case provide the full cookie path using the -p <cookie_path> option.

1. If the cookie is in a different home directory you can use the parameter -h <cookie_home> to provide an alternative home directory.

1. If you know the cookie you can provide the cookie directly via the option -c <cookie_value>

### Short and long names

Currently the script is only using `-sname`. Even though there is an -l option this is currently not functional. **TODO**

### EPMD Port

The default epmd port is 4369. Unfortunately the remote epmd port and locally mapped epmd port need to be identical. So in the default setup `kubernetes_observer` will kill any locally running epmd service before forwarding its port, so that the Elixir/Erlang remote shell is opened against the remote node and not locally.
