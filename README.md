UCloud K8s
===

Welcome to the ucloud-k8s deployment guide! 🎉 This guide will walk you through the essential steps to set up your Kubernetes cluster, install Headscale, and deploy the Pulumi stacks on UCloud with ease. Let's dive in!

## 📋 Requirements

Before diving into the deployment, you'll need to perform some initial setup steps. Once these are completed, the rest of the process should be smooth sailing. 🌊

### 🛠️ Tools

Since the platform is deployed on VMs, you only need to install the deployment tools, not all the tools needed during the deployment. These are installed as part of the deployment process on the servers that need them. Please install the following tools:

* **[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html):** An open-source automation tool for configuration management, application deployment, and task automation.
* **[Terraform](https://opentofu.org/docs/intro/install/):** An infrastructure as code tool that allows you to build, change, and version infrastructure safely and efficiently.

### 📄 Secret Files

A lot of technologies are used, and all have their own way of doing authentication. For you, this means that you will have to create a few credential files. Please follow the instructions below:

- **SSH Key:** You'll need a key to access UCloud machines via SSH. This key's location is specified in `ansible/ansible.cfg`. Ensure you have generated an SSH key pair and placed the private key in the specified location.
- **Tailscale Auth Key:** Generate an auth key from Tailscale and place it in a file named `tailscale_key` at the root level of this repository. This key is used to authenticate your machines with the Tailscale network. If needed, you can alter the key location in the `connect to tailscale` task within the `tailscale` role.
- **hosts.ini File:** Create a `hosts.ini` file in the `ansible` directory. This file defines the inventory of hosts for Ansible to manage. Refer to the `hosts.ini.example` file for the required roles and format.
- **Pulumi Passphrase File:** Prepare a `pulumi_passphrase` file at the root level of the repository. This passphrase is used to encrypt and decrypt your Pulumi stack secrets during deployment.
- **Terraform Credentials:** You will need to create a service account and generate a `credentials.json` file for Terraform. This file should be located in `terraform/credentials.json`. Follow this [guide](https://statics.teams.cdn.office.net/evergreen-assets/safelinks/1/atp-safelinks.html) to generate the credentials.

By following these steps, you'll ensure that all necessary tools and credentials are in place for a successful deployment.

---

You can generate an SSH key using the command:

```bash
ssh-keygen -t ed25519
```

And you can generate a Tailscale auth key by executing the following command on the GCP instance:
```bash
sudo headscale --user ctf preauthkeys create --ephemeral --reusable
```

The Pulumi passphrase you just need to know 😉

## ✅ Deploy workflow

With everything in place, you're ready to bring your Kubernetes cluster to life. Here's how:

1. Execute the Terraform script in `terraform` using `tofu apply`.
2. Configure Headscale and NGINX on GCP by running the Ansible script in `ansible-gcp` using `ansible-playbook cluster.yml`.
3. Configure the Kubernetes cluster and deploy Pulumi stacks by running the Ansible script in `ansible` using `ansible-playbook cluster.yml`.

**Important:** NGINX only resolves hostnames to IP addresses once or when reloaded. This means that once you initially install NGINX, the UCloud servers won't be ready. Only after the UCloud servers have connected to Tailscale can you reload the NGINX configuration to allow TCP streaming to the servers. This step must be performed before deploying the last Pulumi stack since the deployment depends on being reachable from the domain name.

## 💣 Destroy Resources
Don't worry too much about the UCloud servers. You can easily shut them down using the provided UI, and everything will be deleted. 🎉

However, you need to be more cautious with the Terraform script, as we have a static IP address that we want to protect for our domain. Please use the following script to safely destroy those resources:

```bash
tofu apply -destroy -target="google_compute_network.ctf_network" -target="google_compute_firewall.ssh_rule" -target="google_compute_firewall.headscale_enabled" -target="google_compute_instance.headscale_core_ctf" -target="local_file.hosts" -target="local_file.headscale_config"
```

Once these resources have been destroyed, it's as if it never happened. ✨ What a fantastic opportunity for a fresh, clean deploy!

## ❓FAQ
Sometimes things might go awry during deployment, but worry not! We've compiled a list that might help solve errors based on their descriptions.

### 🔄 The Headscale Magic DNS Has Become Stale
It's possible that the Magic DNS Headscale uses might resolve hostnames to old IP addresses. While this isn't a common issue, it has happened before. Last time, renaming a host solved it. Since NGINX expects a specific name, it's recommended to rename a host to its current hostname. Any command that refreshes the DNS should be fine.

### 🔑 Initialization Error on Step Login
Keycloak and Step CA have a circular dependency. Keycloak needs a TLS certificate to start, but Step needs Keycloak for its OIDC provisioner initialization. A local command in Pulumi handles this locally, and there's a step in Ansible for production. If Step is not correctly initialized, ensure the cluster is reachable through the domain. Once confirmed, restart Step using the command:

```bash
kubectl rollout restart -n ucloud statefulset step-step-certificates
```

### 🔐 Broken TLS Certificate
If a certificate has gone bad for some unknown reason, it might be sufficient to simply delete the affected pod, as Step will then issue a completely new TLS certificate. Sometimes the certificate is provided as a secret, other times directly injected into the pod. If deleting the pod doesn't work, try deleting the TLS secret, remove the secret from the Pulumi state, and run `pulumi up` to recreate the secret. The certificate manager and sidecar containers should renew certificates automatically, so this shouldn't be a problem.

If NGINX on GCP isn't forwarding traffic correctly, it's impossible to get a trusted Let's Encrypt certificate since Let's Encrypt expects to communicate with Certbot on port 80. If the homepage doesn't work, this might be the issue. Fix the proxy on GCP, delete the SSLH pod, and try again after some time.