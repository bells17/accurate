# Sub-namespace operations

Sub-namespaces is a feature of Accurate to allow tenant users to create Namespaces and delete the created Namespaces.

Sub-namespaces can be created under either a root Namespace or a sub-namespace.

In the following examples, `<name>` represents a Namespace name to be changed.
Likewise, `<parent>` represents a root or another sub-namespace.

## Setting a Namespace as a root Namespace

Suppose that Accurate is configured to propagate `team` label.

Using `kubectl accurate`:

```console
$ kubectl accurate ns set-type <name> root
$ kubectl label ns <name> team=foo
```

Applying YAML manifests:

```console
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
  labels:
    accurate.cybozu.com/type: root
    team: foo
```

### Preparing resources for tenant users

In almost all cases, a root Namespace should have RoleBinding for a group of tenant users.
The RoleBinding should be annotated with `accurate.cybozu.com/propagate=update`.

```console
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: <name>
  name: admin
  annotations:
    accurate.cybozu.com/propagate: update
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: Group
  name: foo
  apiGroup: rbac.authorization.k8s.io
```

## Reverting a root Namespace to a normal one

Using `kubectl accurate`:

```console
$ kubectl accurate ns set-type <name> none
```

Applying YAML manifests:

Remove `accurate.cybozu.com/type` label.

## Creating a sub-namespace

Using `kubectl accurate`:

```console
$ kubectl accurate sub create <name> <parent>
```

Applying YAML manifests:

```console
apiVersion: accurate.cybozu.com/v1
kind: SubNamespace
metadata:
  namespace: <parent>
  name: <name>
```

## Deleting a created sub-namespace

Using `kubectl accurate`:

```console
$ kubectl accurate sub delete <name>
```

Applying YAML manifests:

Delete the created SubNamespace object.

## Changing the parent of a sub-namespace

Only cluster admins can do this.

Using `kubectl accurate`:

```console
$ kubectl accurate sub move <name> <new-parent>
```

Applying YAML manifests:

```console
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
  labels:
    accurate.cybozu.com/parent: <new-parent>
```

## Converting a normal Namespace to a sub-namespace

Only cluster admins can do this.

Using `kubectl accurate`:

```console
$ kubectl accurate sub graft <name> <parent>
```

Applying YAML manifests:

```console
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
  labels:
    accurate.cybozu.com/parent: <parent>
```

## Converting a sub-namespace to a root Namespace

Only cluster admins can do this.

Using `kubectl accurate`:

```console
$ kubectl accurate sub cut <name>
```

Applying YAML manifests:

```console
apiVersion: v1
kind: Namespace
metadata:
  name: <name>
  labels:
    accurate.cybozu.com/type: root
    # and remove accurate.cybozu.com/parent label
```
