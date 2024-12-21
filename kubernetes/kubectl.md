- config :
    context : is a set of parameters that describe how to connect to a cluster
  - kubectl config get-contexts
  - kubectl config use-context
  - kubectl config current-context
  
- commands:
  - cluster-info : show cluster info
  - get {resource} : list resources
  - explain {resource} : show the documentation for a resource
  - describe {resource} : show detailed information about a resource
  - logs {pod-name} : show the logs of a pod
  - exec {pod-name} -- {command} : execute a command in a pod
  - delete {resource} {resource-name} : delete a resource
  - edit {resource} {resource-name} : edit a resource, e.g. a pod 