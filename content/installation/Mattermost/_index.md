+++
title = "Mattermost"
date = 2019-01-04T16:16:15+05:30
draft = false
weight = 20
toc = true
+++

<h3 class="section-head" id="h-install-BotKube-mattermost"><a href="#h-install-BotKube-mattermost">Install BotKube to the Mattermost team</a></h3>

<p>Follow the steps below to install BotKube in your Mattermost Team (v5.14.0). </p>

#### 1. Enable Personal Access Token
Login with System Admin account, and in the Menu proceed to **System console > Integrations > Integration Management** and enable **Personal Access Token**.

![mm_token_access](/images/mm_token_access.png)

#### 2. Create BotKube user
To create a BotKube user, if not already created, proceed to menu and Get team invite link. Logout from admin account and paste the link in the address bar and create a user with the username **BotKube**.

![mm_botkube_user](/images/mm_botkube_user.png)

#### 3. Manage Roles for BotKube user
Login as System Admin, in the Menu proceed to **System console > Users**. For BotKube user, Manage Roles and allow tokens and post_all access.

![mm_botkube_roles](/images/mm_botkube_roles.png)

#### 4. Create Token for BotKube user
Login as BotKube user, in the Menu proceed to **Account Settings > Security > Personal Access Token > Create** and save the token.

![mm_botkube_token](/images/mm_botkube_token.png)

#### 5. Add BotKube to a channel
Add BotKube user created to the channel you want to receive notifications in.

<h3 class="section-head" id="h-install-BotKube-slashcommand"><a href="#h-install-BotKube-slashcommand">Configure /botkubehelp Slash Command</a></h3>

**1.** First, go to **Main Menu > Integrations > Slash Commands**. (If you don’t have the Integrations option in your Main Menu, slash commands may not be enabled on your Mattermost server or may be disabled for non-admins. Enable them from System Console > Integrations > Custom Integrations in prior versions)

**2.** Click **Add Slash Command** and add following details for the command and click **Save**.

Field | Value
--- | ---
Title | BotKube 
Description | Show BotKube help
Command Trigger Word | botkubehelp
Request URL | https://botkube.herokuapp.com/help
Request Method | POST
Autocomplete | True

![mm_botkube_slash_cmd](/images/mm_botkube_slash_cmd.png)

**3.** Verify executing **/botkubehelp** in a channel.

![mm_botkube_help](/images/mm_botkube_help.png)


<h3 class="section-head" id="h-install-BotKube-k8s"><a href="#h-install-BotKube-k8s">Install BotKube in Kubernetes cluster</a></h3>

<h4>BotKube install: Using helm</h4>

- We will be using [helm](https://helm.sh/) to install BotKube in Kubernetes. Follow [this](https://docs.helm.sh/using_helm/#installing-helm) guide to install helm if you don't have it installed already
- Add **infracloudio** chart repository

```bash
$ helm repo add infracloudio https://infracloudio.github.io/charts
```

- Deploy BotKube backend using **helm install** in your cluster.

```bash
$ helm repo update
$ helm install --version v0.9.0 --name botkube --namespace botkube \
--set config.communications.mattermost.enabled=true \
--set config.communications.mattermost.url=<MATTERMOST_SERVER_URL> \
--set config.communications.mattermost.cert=<MATTERMOST_CERT> \
--set config.communications.mattermost.token=<MATTERMOST_TOKEN> \
--set config.communications.mattermost.team=<MATTERMOST_TEAM> \
--set config.communications.mattermost.channel=<MATTERMOST_CHANNEL> \
--set config.settings.clustername=<CLUSTER_NAME> \
--set config.settings.allowkubectl=<ALLOW_KUBECTL> \
--set image.repository=infracloudio/botkube \
--set image.tag=v0.9.0 \
infracloudio/botkube
```

where,<br>
- **MATTERMOST_SERVER_URL** is the URL (including http/https schema) where Mattermost is running<br>
- **MATTERMOST_CERT** is the SSL certificate file for HTTPS connection. Place it in Helm directory and specify the path<br>
- **MATTERMOST_TOKEN** is the Token received by creating Personal Access Token for BotKube user<br>
- **MATTERMOST_TEAM** is the Team name where BotKube is added<br>
- **MATTERMOST_CHANNEL** is the Channel name where BotKube is added and used for communication<br>
- **CLUSTER_NAME** is the cluster name set in the incoming messages<br>
- **ALLOW_KUBECTL** set true to allow kubectl command execution by BotKube on the cluster<br>

- To deploy with TLS, replace **MATTERMOST_CERT** with the location of SSL certificate file placed in Helm directory. Leave this value to None if deploying without TLS.

   Configuration syntax is explained [here](/configuration).

Send **@BotKube ping** in the channel to see if BotKube is running and responding.

{{% notice note %}}
  With default configuration, BotKube will watch all the resources in all the namespaces for _create_, _delete_ and _error_ events.<br>
  If you wish to monitor only specific resources, follow the steps given below:
{{% /notice%}}

  - Create new file config.yaml and add resource configuration as described on the [configuration](/configuration) page.

    (You can refer sample config from https://github.com/infracloudio/botkube/tree/develop/helm/botkube/sample-res-config.yaml)

    ```
    config:
      ## Resources you want to watch
      resources:
        - name: pod                # Name of the resources e.g pod, deployment, ingress, etc.
          namespaces:              # List of namespaces, "all" will watch all the namespaces
            include:
              - all
            ignore:
              - kube-system
          events:                  # List of lifecycle events you want to receive, e.g create, update, delete, error OR all
            - create
            - delete
            - error
        - name: job
          namespaces:
            include:
              - all
            ignore:
              - dev
          events:
            - create
            - update
            - delete
            - error
    ```
  - Pass the yaml file as a flag to `helm install` command.
    e.g

    ```
    helm install --version v0.9.0 --name botkube --namespace botkube -f /path/to/config.yaml --set=...other args..
    ```

  Alternatively, you can also update the configuration at runtime as documented [here](/configuration/#updating-the-configuration-at-runtime)


<h4>Using kubectl</h4>

- Make sure that you have kubectl cli installed and have access to Kubernetes cluster
- Download deployment specs yaml

```bash
$ wget -q https://raw.githubusercontent.com/infracloudio/botkube/master/deploy-all-in-one.yaml
```

- Open downloaded **deploy-all-in-one.yaml** and update the configuration.<br>
Set *MATTERMOST_ENABLED*, *MATTERMOST_SERVER_URL*, *MATTERMOST_TOKEN*, *MATTERMOST_TEAM*, *MATTERMOST_CHANNEL*, *clustername*, *allowkubectl* and update the resource events configuration you want to receive notifications for in the configmap.<br>

where,<br>
- **MATTERMOST_ENABLED** set true to enable Mattermost support for BotKube<br>
- **MATTERMOST_SERVER_URL** is the URL where Mattermost is running<br>
- **MATTERMOST_TOKEN** is the Token received by creating Personal Access Token for BotKube user<br>
- **MATTERMOST_TEAM** is the Team name where BotKube is added<br>
- **MATTERMOST_CHANNEL** is the Channel name where BotKube is added and used for communication<br>
- **clustername** is the cluster name set in the incoming messages<br>
- **allowkubectl** set true to allow kubectl command execution by BotKube on the cluster<br>

   Configuration syntax is explained [here](/configuration).

- Create **botkube** namespace and deploy resources

```bash
$ kubectl create ns botkube && kubectl create -f deploy-all-in-one.yaml -n botkube
```

- Check pod status in botkube namespace. Once running, send **@BotKube ping** in the Slack channel to confirm if BotKube is responding correctly.

- To deploy with TLS, download and use deploy-all-in-one-tls.yaml. Replace **ENCODED_CERTIFICATE** with your base64 encoded certificate value in the secret. To get a base64 encoded value of your certificate, use below command and replace <YOUR_CERTIFICATE> with the certificate name.

```bash
$ cat <YOUR_CERTIFICATE> | base64 -w 0 
```

<br>
<h3 class="section-head" id="h-uninstall-BotKube-mattermost"><a href="#h-uninstall-BotKube-mattermost">Remove BotKube from Mattermost Team</a></h3>
- Deactive or remove BotKube user from Mattermost Team. Login as System Admin, in the Menu proceed to System console -> Users ->  Deactivate<br>
- Archive Channel created for BotKube communication if required.

<h3 class="section-head" id="h-uninstall-BotKube-k8s"><a href="#h-uninstall-BotKube-k8s">Remove BotKube from Kubernetes cluster</a></h3>
<h4>Using helm</h4>
<p>If you have installed BotKube backend using **helm**, execute following command to completely remove BotKube and related resources from your cluster</p>

```bash
$ helm delete --purge botkube
```

<h4>BotKube install: Using kubectl</h4>

```bash
$ kubectl delete -f https://raw.githubusercontent.com/infracloudio/botkube/master/deploy-all-in-one.yaml -n botkube
```
