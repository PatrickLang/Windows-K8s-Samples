# Setting up a manual cluster with ACS-engine

Starting point: https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md

I did all the steps here using [Ubuntu from the Windows Store](https://www.microsoft.com/en-us/store/p/ubuntu/9nblggh4msv6).


Get latest build - see https://github.com/Azure/acs-engine/releases/

```bash
curl -L https://github.com/Azure/acs-engine/releases/download/v0.15.2/acs-engine-v0.15.2-linux-amd64.tar.gz | tar xvfz -
sudo mv acs-engine-v0.15.2-linux-amd64/acs-engine /usr/local/bin
rm -rf acs-engine-v0.15.2-linux-amd64/
```

Get SSH public key

```bash
ls ~/.ssh/id_rsa.pub

```


Generate Service Principal

```bash
export SUBSCRIPTION_ID=`az account show | jq .id | sed "s/\"//"g`
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}" > secret.json
```

Get template

```bash
curl -o kubernetes.json https://raw.githubusercontent.com/Azure/acs-engine/master/examples/windows/kubernetes.json
```

Now, let's fill in the hard stuff automatically. This will add some data to the template and create `kubernetes-secret.json`. Don't share that file!

- Your SSH public key from `.ssh/id_rsa.pub`
- The service account credentials stored in `secret.json`

```bash
export APPID=`jq '.appId' secret.json`
export APPPASS=`jq '.password' secret.json`
cat kubernetes.json | jq ".properties.linuxProfile.ssh.publicKeys[0].keyData = \"`cat ~/.ssh/id_rsa.pub`\"" | jq ".properties.servicePrincipalProfile.clientId = $APPID" | jq ".properties.servicePrincipalProfile.secret = $APPPASS" > kubernetes-secret.json
```


Now, update it

- Change username & passwords
- Add a dnsprefix
- Choose VM size (hint: `az vm list-sizes --location westus2 -otable`)
- Add `orchestratorVersion` for the version you want

```
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "orchestratorVersion": "1.10.1"
    },
```

Generate the Azure Resource Manager template

```
$ acs-engine generate ./kubernetes-secret.json

INFO[0000] Generating assets into _output/plang0308...
```

This will generate a directory named with the dnsPrefix in the template file - `plang0308` in this case


Create a resource group

```
export rgname=plang0308rg
export name=plang0308
az group create --name $rgname --location westus2
```

Deploy it

```
az group deployment create --name $name --resource-group $rgname --template-file ./_output/$name/azuredeploy.json  --parameters ./_output/$name/azuredeploy.parameters.json
```

After several minutes, it will return a whole bunch of JSON. Look for `masterFQDN`.

```
      "masterFQDN": {
        "type": "String",
        "value": "plang0308.westus2.cloudapp.azure.com"
      },
```

SSH to that FQDN with the username you put in the linuxProfile section of `kubernetes-secret.json` :

```
ssh patrick@plang0308.westus2.cloudapp.azure.com
```

Now, you can verify the cluster is up with `kubectl get node`

If you want to manage this from your local machine, go ahead and disconnect that SSH session. Then copy the kube config to your local machine in a new directory, and set `KUBECONFIG`

```
mkdir ~/.$name
export KUBECONFIG=~/.$name/config
ssh patrick@plang0308.westus2.cloudapp.azure.com 'cat ~/.kube/config' >> $KUBECONFIG
kubectl get node
```


## Enabling Hyper=V

With Hyper-V isolation, you can run containers using the same, or older Windows Server versions. For example, you can run a container built with the Windows Server 2016 base image such as `microsoft/iis:windowsservercore-ltsc2016` on a Windows Server version 1709 node. It could also run alongside another pod with containers built against Windows Server version 1709 such as `microsoft/iis:windowsservercore-1709`. For more details, see the [version compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility) article on docs.microsoft.com



Steps needed:

- Prerequisite: Ensure the Hyper-V role is enabled, and the processor (and virtualization solution) has virtualization support enabled. 
- For Windows Server version 1709 - the April 2018 cumulative update ([KB4093112](https://support.microsoft.com/en-us/help/4093112/windows-10-update-kb4093112) / version 10.10.16299.371) or later
- Add flag to `kubectl.exe` startup options
- Add annotation to deployment config

### Opening feature-gates

Here's an example of the feature gate that's needed. The other options should be kept as-is in the kubelet startup script.

```
c:\k\kubelet.exe --pod-infra-container-image=kubletwin/pause ... --feature-gates HyperVContainer=true
```

> Note: Check this [acs-engine issue](https://github.com/Azure/acs-engine/issues/2627) - once resolved the next steps will be much easier.

This is easy to do before you create a cluster using acs-engine by:

1. Running `acs-engine generate ...` as above
  - Be sure to use Dv3/Ev3 series VMs
2. Don't run `az group deploy` yet, wait
3. Modify `output/<dnsprefix>/azuredeploy.json` to add `--feature-gates ...`


### Deploying a pod with Hyper-V isolation

In the pod deployment, be sure to add the annotation `experimental.windows.kubernetes.io/isolation-type: hyperv`, and use a tag to ensure you're picking the intended Windows version if there are multiple available.

(_See whoami-2016.yaml and whoami-1709.yaml_ in this same repo)

## TODOs

Cleanup steps - remove deployments like [this](https://github.com/Azure/acs-engine/blob/34803633e92b1beef0ca7585cb2aa03a90d40f47/test/e2e/cleanup.sh#L46) to avoid failures deleting the group
