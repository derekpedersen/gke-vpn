# GKE VPN

Here is the setup I used to deploy an openvpn server to gke

```bash
mkdir ovpn0 && cd ovpn0
```

``` bash
docker run --net=none --rm -it -v $PWD:/etc/openvpn kylemanna/openvpn ovpn_genconfig \
    -u udp://VPN.DEREKPEDERSEN.COM:31304 \
    -C 'AES-256-GCM' -a 'SHA384' -T 'TLS-ECDHE-ECDSA-WITH-AES-256-GCM-SHA384' \
    -b -n 185.121.177.177 -n 185.121.177.53 -n 87.98.175.85
```

```bash
docker run -e EASYRSA_ALGO=ec -e EASYRSA_CURVE=secp384r1 \
    --net=none --rm -it -v $PWD:/etc/openvpn kylemanna/openvpn ovpn_initpki
```

```bash
docker run --net=none --rm -it -v $PWD:/etc/openvpn kylemanna/openvpn ovpn_copy_server_files
```

```bash
export CLIENTNAME="minty"
```

```bash
docker run -e EASYRSA_ALGO=ec -e EASYRSA_CURVE=secp384r1 \
    --net=none --rm -it -v $PWD:/etc/openvpn kylemanna/openvpn easyrsa build-client-full $CLIENTNAME
```

```bash
docker run --net=none --rm -v $PWD:/etc/openvpn kylemanna/openvpn ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
```

```bash
sudo chown -R $USER:$USER server/*
```

```bash
kubectl create secret generic ovpn0-key --from-file=server/pki/private/VPN.DEREKPEDERSEN.COM.key
```

```bash
kubectl create secret generic ovpn0-cert --from-file=server/pki/issued/VPN.DEREKPEDERSEN.COM.crt
```

```bash
kubectl create secret generic ovpn0-pki \
    --from-file=server/pki/ca.crt --from-file=server/pki/dh.pem --from-file=server/pki/ta.key
```

```bash
kubectl create configmap ovpn0-conf --from-file=server/
```

```bash
kubectl create configmap ccd0 --from-file=server/ccd
```

```bash
kubectl apply -f ./deployment.yaml
```

```bash
gcloud compute firewall-rules create ovpn0 --allow=udp:31304
```