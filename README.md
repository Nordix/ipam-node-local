# ipam-node-local

A "meta" CNI-plugin for assigning addresses to PODs on a
[Kubernetes](https://kubernetes.io/) (k8s) cluster. `Ipam-node-local`
uses the "standard" ipam
[host-local](https://github.com/containernetworking/plugins/tree/master/plugins/ipam/host-local)
but make sure that the addresses are taken from the ranges assigned to
the node by K8s.

At the moment `ipam-node-local` is hard-wired for testing the k8s
dual-stack PR
[#73977](https://github.com/kubernetes/kubernetes/pull/73977).  This
means that the ipv6 range is assumed to be /24 and the `podCIDRs` item
is assumed to exist in the k8s node objects.


## Usage

Copy the `node-local` script to the /opt/cni/bin directory. Note that
`host-local` must also be installed here;

```
# ls /opt/cni/bin/
bridge*     host-local* loopback*   node-local*
```

Configure your CNI-plugin to use ipam `node-local`. Example;


```
{
        "cniVersion": "0.3.1",
        "name": "xcluster",
        "type": "bridge",
        "bridge": "cbr0",
        "isGateway": true,
        "isDefaultGateway": true,
        "ipam": {
                "type": "node-local"
        }
}
```

## Local testing

The `node-local` script can be tested off-line;

```
cat > /tmp/podCIDR <<EOF
11.0.3.0/24
1000:300::/24
EOF
HOSTNAME=vm-003 HOST_LOCAL=cat ./node-local < ./example-cni.conf
```

This will print the "real" cni configuration that will be passed to
`host-local`;

```
{
  "cniVersion": "0.3.1",
  "name": "xcluster",
  "type": "bridge",
  "bridge": "cbr0",
  "isGateway": true,
  "isDefaultGateway": true,
  "ipam": {
    "type": "host-local",
    "ranges": [
      [
        {
          "subnet": "11.0.2.0/24",
          "gateway": "11.0.2.1"
        }
      ],
      [
        {
          "subnet": "1100:200::/24",
          "gateway": "1100:200::1"
        }
      ]
    ]
  }
}
```

