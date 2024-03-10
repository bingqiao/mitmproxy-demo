# Demo of mitmproxy
This is a demo project/tutorial for using [mitmproxy](https://mitmproxy.org) in both regular and reverse modes deployed via docker-compose or in a Kuternetes cluster.

# Prerequisites
* [Docker Desktop](https://docs.docker.com/engine/install/)
* Kubernetes tools and cluster, if you'd like to try Kubernetes deployment. This project is tested using [kind](https://kind.sigs.k8s.io). This link has infomration on installing both `kubectl` and `kind`: [Install Tools](https://kubernetes.io/docs/tasks/tools/)
* Code editor such as [Visual Studio Code](https://code.visualstudio.com/download)

# Projects
* 0-wttr: a simple golang app `wttr` that makes call to https://wttr.in for weather report
* 1-mitmproxy-reverse: `wttr` proxied by `mitmproxy` in reverse mode for incoming requests
* 2-mitmproxy-regular: `wttr` proxied by `mitmproxy` for outgoing requests
* 3-mitmproxy-both: `wttr` proxied by two mitmproxy instances for both incoming and outgoing requestss

N.B Kubernetes releated tutorial assumes a local cluster named `k8s-dev` has been created using [kind](https://kind.sigs.k8s.io).
