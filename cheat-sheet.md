
## Commands

### `kubectl version`

Display:
- Client Version (version of the cli package)
- Server Version

### `kubectl config`

Modify kubeconfig files using subcommands like "kubectl config set current-context my-context"

The loading order follows these rules:
  1. `--kubeconfig` flag
  2. If `$KUBECONFIG` environment variable is set
  3. `${HOME}/.kube/config`

Available Commands:
- `current-context` Displays the current-context
- `delete-cluster`  Delete the specified cluster from the kubeconfig
- `delete-context`  Delete the specified context from the kubeconfig
- `get-clusters`    Display clusters defined in the kubeconfig
- `get-contexts`    Describe one or many contexts
- `rename-context`  Renames a context from the kubeconfig file.
- `set`             Sets an individual value in a kubeconfig file
- `set-cluster`     Sets a cluster entry in kubeconfig
- `set-context`     Sets a context entry in kubeconfig
- `set-credentials` Sets a user entry in kubeconfig
- `unset`           Unsets an individual value in a kubeconfig file
- `use-context`     Sets the current-context in a kubeconfig file
- `view`          Display merged kubeconfig settings or a specified kubeconfig file

### `kubectl get <resource>`

Display one or many resources

- Uninitialized objects are not shown unless `--include-uninitialized` is passed.
- By specifying the output as 'template' and providing a Go template as the value of the --template flag, you can filter the attributes of the fetched resources.
- Use "kubectl api-resources" for a complete list of supported resources.

Options:
- `-A`, `--all-namespaces=false`
- `--chunk-size=500`
- `--field-selector key1=value1,key2=value2` (supports '=', '==', and '!=')
- `-f`, `--filename=[]`
- `--ignore-not-found=false`
- `-k`, `--kustomize=''`
- `-L`, `--label-columns=[]`
- `--no-headers=false`
- `-o`, `--output=json|yaml|wide|name|custom-columns=...|custom-columns-file=...|go-template=...|go-template-file=...|jsonpath=...|jsonpath-file=...` 
- `--output-watch-events=false`
- `--raw=''`: Raw URI to request from the server.  Uses the transport specified by the kubeconfig file.
- `-R`, --recursive=false
- `-l`, `--selector=''`
- `--server-print=true`: If true, have the server return the appropriate table output. Supports extension APIs and CRDs.
- --show-kind=false
- --show-labels=false
- --sort-by='': (the field specification is expressed as a JSONPath expression - e.g. '{.metadata.name}')
- --template='': Template string or path to template file to use when -o=go-template, -o=go-template-file. The template format is golang templates [http://golang.org/pkg/text/template/#pkg-overview].
- -w, --watch=false
- --watch-only=false: Watch for changes to the requested object(s), without listing/getting first.


Use "kubectl options" for a list of global command-line options (applies to all commands).


### `kubectl describe <resource> <name>`

Show details of a specific resource or group of resources

Options:
- `-A`, `--all-namespaces=false`
- `-f`, `--filename=[]`
- `-k`, `--kustomize=''` (can't be used together with -f or -R)
- `-R`, `--recursive=false`
- `--show-events=true`: If `true`, display events related to the described object.
