https://github.com/cilium/cilium-cli/releases

VERSION=v0.15.5
curl -sLO "https://github.com/cilium/cilium-cli/releases/download/$VERSION/cilium-linux-amd64.tar.gz"
tar xvf cilium-linux-amd64.tar.gz 
mv cilium /usr/local/bin/
