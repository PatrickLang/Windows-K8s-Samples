# Windows Kubernetes Samples

This is a group of sample apps that I use to build demos. Check the history to get an idea of when each was used last.


## IIS

This is a quick sample to deploy a web server in a replicaset with a loadbalancer. You can use it as a quick first test that covers end to end networking and scale up / down.

(This assumes Windows Server version 1803, there are samples for other versions as well in this folder)

1. Create the deployment with `kubectl apply -f iis/iis-1803.yaml`
2. Wait for the service to have an IP by checking `kubectl get svc` repeatedly
3. Visit `http://<external ip>`
4. Scale it up with `kubectl scale deploy/iis-1803 --replicas=2`

If you are on a node in the cluster, you can check them all with
`kubectl get pods -o=jsonpath='{range .items[*]}{.status.podIP}{"\n"}{end}' | xargs -n1 curl > /dev/null`


## RazorPagesMovie

This is based on the samples from https://github.com/aspnet/Docs, specifically https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie . It's a straightforward Razor based application that uses SQL or SQLite for the backing database. The `appconfig.json` in this repo is preconfigured for SQLLite so that only a single container is needed. You can use this to test persistent volume claims by mapping a PV to c:\data.

### Build & test locally

You can build and test this on a Windows 10 machine with Docker for Windows installed. No development tools are needed since the build is run inside a container.

Build it - `docker build --pull -t razorpagesmovie .`

- Alternatively you could `docker pull patricklang/razorpagesmovie:aspnet-2.0.5-nanoserver-1709` instead, and use that in the following steps

Test it - `docker run --name razorpagesmovie --rm -it -p 8000:80 razorpagesmovie`, then browse to (http://127.0.0.1:8000/Movies)

Try it with a local volume - `docker run --name razorpagesmovie --rm -it -p 8000:80 -v $PWD\data:c:\app\data\ razorpagesmovie`. Browse to it, make some changes, then try it again. Changes are persisted, and you can see the SQLite database file at .\data\MvcMovie.db on your machine.



### Deploy in Kubernetes with Flexvolume

> TODO - the volume steps aren't done yet

`kubectl create -f movies.yaml`


## FabrikamFiber

This demo includes Visual Studio projects, and needs the following to build:

- Docker for Windows
- Visual Studio 2017

It's currently maintained in the [patricklang/fabrikamfiber](https://github.com/PatrickLang/fabrikamfiber/tree/k8s-support2) repo.


## Simple web apps using Hyper-V isolation

[HyperVExamples](./HyperVExamples) has more details on how to enable the alpha Hyper-V support in Kubernetes 1.10 and later. It includes examples of deploying the same app that was built for 3 different OS versions. Without Hyper-V isolation, only one of the three will work. With Hyper-V isolation, all 3 can run side by side on Windows Server version 1803.


## Random Tips

- Greg Bayer has a good blog post on [Moving FIles from one Git Repository to Another, Preserving History](https://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/) which I used to seed this repo with just the sample code, instead of the full .Net doc set :)
