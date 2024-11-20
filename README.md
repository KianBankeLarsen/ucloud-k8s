UCloud K8s
===

Welcome to the **ucloud-k8s** deployment guide! This guide will take you through the essential steps to set up your Kubernetes cluster, install Tailscale, and deploy the Pulumi stacks on UCloud with ease. Let's get started!

## 📋 Requirements

Before you begin, make sure you have these three essentials:

- **SSH Key:** You'll need a key to access UCloud machines via SSH. This key's location is specified in `ansible/ansible.cfg`.
- **Tailscale Auth Key:** Secure an auth key from [Tailscale](https://tailscale.com/kb/1085/auth-keys) and place it in a file named `tailscale_key` at the root level of this repository. This can be altered in the `connect to tailscale` task within the `tailscale` role if needed.
- **hosts.ini File:** Create a `hosts.ini` file in the `ansible` directory. Check the `hosts.ini.example` file for the required roles and format.
- **Pulumi Passphrase File:** Prepare a `pulumi_passphrase` file at the root level of the repository, used in the Pulumi deployment tasks.

## 🚀 Running the Playbook

With everything in place, you're ready to bring your Kubernetes cluster to life. Here's how:

1. Navigate to the `ansible` directory where the playbook is located.
2. Run the playbook by executing the following command:

    ```bash
    ansible-playbook cluster.yml
    ```

By following these steps, you will efficiently set up your Kubernetes cluster, install Tailscale, and deploy the Pulumi stacks. This ensures a seamless deployment process, making your platform operational in no time. Happy deploying! 🚀