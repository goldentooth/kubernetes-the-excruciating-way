# Fixing MetalLB

As mentioned [here](./027_welcome_back.md), I purchased a new router to replace a power-hungry Dell server running OPNsense, and that cost me BGP support. This kills my MetalLB configuration, so I need to switch it to use Layer 2.

That shouldn't be too bad.

I think it's just a matter of deleting the BGP advertisement:

```bash
$ sudo kubectl -n metallb delete BGPAdvertisement primary
bgpadvertisement.metallb.io "primary" deleted
```

and creating an L2 advertisement:

```bash
$ cat tmp.yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: primary
  namespace: metallb

$ sudo kubectl apply -f tmp.yaml
l2advertisement.metallb.io/primary created
```

After adding the static route to my router, I can see the friendly `go-httpbin` response when I navigate to https://10.4.11.1/
