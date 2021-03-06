本文档基于K8S 1.10

* A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.
* NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods

Prerequisites

Network policies are implemented by the network plugin, so you must be using a networking solution which supports `NetworkPolicy` - simply creating the resource without a controller to implement it will have no effect.

An exampleNetworkPolicymight look like this:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

_POSTing this to the API server will have no effect unless your chosen networking solution supports network policy._

So, the example NetworkPolicy:

* isolates “role=db” pods in the “default” namespace for both ingress and egress traffic \(if they weren’t already isolated\)
* allows connections to TCP port 6379 of “role=db” pods in the “default” namespace from any pod in the “default” namespace with the label “role=frontend”
* allows connections to TCP port 6379 of “role=db” pods in the “default” namespace from any pod in a namespace with the label “project=myproject”
* allows connections to TCP port 6379 of “role=db” pods in the “default” namespace from IP addresses that are in CIDR 172.17.0.0/16 and not in 172.17.1.0/24
* allows connections from any pod in the “default” namespace with the label “role=db” to CIDR 10.0.0.0/24 on TCP port 5978

something：

* spec:NetworkPolicyspec has all the information needed to define a particular network policy in the given namespace.

* podSelector: EachNetworkPolicyincludes apodSelectorwhich selects the grouping of pods to which the policy applies. The example policy selects pods with the label “role=db”. An emptypodSelectorselects all pods in the namespace.

* policyTypes: EachNetworkPolicyincludes apolicyTypeslist which may include eitherIngress,Egress, or both. ThepolicyTypesfield indicates whether or not the given policy applies to ingress traffic to selected pod, egress traffic from selected pods, or both. If nopolicyTypesare specified on a NetworkPolicy then by defaultIngresswill always be set andEgresswill be set if the NetworkPolicy has any egress rules.

* ingress: EachNetworkPolicymay include a list of whitelistingressrules. Each rule allows traffic which matches both thefromandportssections. The example policy contains a single rule, which matches traffic on a single port, from one of three sources, the first specified via anipBlock, the second via anamespaceSelectorand the third via apodSelector.

* egress: EachNetworkPolicymay include a list of whitelistegressrules. Each rule allows traffic which matches both thetoandportssections. The example policy contains a single rule, which matches traffic on a single port to any destination in10.0.0.0/24.

## Default Policies {#default-policies}

By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. The following examples let you change the default behavior in that namespace.

### Default deny all ingress traffic {#default-deny-all-ingress-traffic}

You can create a “default” isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any ingress traffic to those pods.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

This ensures that even pods that aren’t selected by any other NetworkPolicy will still be isolated. This policy does not change the default egress isolation behavior

### Default allow all ingress traffic {#default-allow-all-ingress-traffic}

If you want to allow all traffic to all pods in a namespace \(even if policies are added that cause some pods to be treated as “isolated”\), you can create a policy that explicitly allows all traffic in that namespace.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  ingress:
  - {}
```

### Default deny all egress traffic {#default-deny-all-egress-traffic}

You can create a “default” egress isolation policy for a namespace by creating a NetworkPolicy that selects all pods but does not allow any egress traffic from those pods.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

This ensures that even pods that aren’t selected by any other NetworkPolicy will not be allowed egress traffic. This policy does not change the default ingress isolation behavior.

### Default allow all egress traffic {#default-allow-all-egress-traffic}

If you want to allow all traffic from all pods in a namespace \(even if policies are added that cause some pods to be treated as “isolated”\), you can create a policy that explicitly allows all egress traffic in that namespace.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all
spec:
  podSelector: {}
  egress:
  - {}
```

### Default deny all ingress and all egress traffic {#default-deny-all-ingress-and-all-egress-traffic}

You can create a “default” policy for a namespace which prevents all ingress AND egress traffic by creating the following NetworkPolicy in that namespace.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

参考资料：

1. [https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)
2. [https://github.com/ahmetb/kubernetes-network-policy-recipes](https://github.com/ahmetb/kubernetes-network-policy-recipes)



