---
title: "Installation"
description: Learn how to install Coder onto your infrastructure.
---

This article walks you through the process of installing Coder onto your
[Kubernetes cluster](kubernetes/index.md).

## Dependencies

Install the following dependencies if you haven't already:

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [helm](https://helm.sh/docs/intro/install/)

**For Production deployments:** set up and use an external
[PostgreSQL](https://www.postgresql.org/docs/12/admin.html) instance to store
data, including environment information and session tokens.

## Creating the Coder Namespace (Optional)

We recommend running Coder in a separate
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/);
to do so, run

```bash
kubectl create namespace coder
```

Next, change the kubectl context to point to your newly created namespace:

```bash
kubectl config set-context --current --namespace=coder
```

## Installing Coder

1. Add the Coder helm repo

   ```bash
   helm repo add coder https://helm.coder.com
   ```

2. Install the helm chart onto your cluster (see the
   [changelog](../changelog/index.md) for a list of Coder versions or run `helm
   search repo coder -l`)

   ```bash
   helm install coder coder/coder --namespace coder
   ```

   **Steps 3-5 are optional for non-production deployments.**

3. Get a copy of your helm chart so that you can modify it; you'll need to
   modify the helm chart to update your PostgreSQL databases (step 4) and enable
   Dev URLs (step 5):

   a. Get a copy of your existing helm chart and save as `values.yaml`: `helm
   show values coder/coder > values.yaml`

   b. Edit the `values.yaml` file as needed. Be sure to remove the lines that
   you are *not* modifying, otherwise the contents of `values.yaml` will
   override those in the default chart.

   c. Upgrade/install your Coder deployment with the updated helm chart (be sure
      to replace the placeholder value with your Coder version): `helm upgrade
      coder coder/coder -n coder --version=<VERSION> -f values.yaml`. **This
      must be done for whenever you update the helm chart.**

4. Add the following to your helm chart so that Coder uses your external
   PostgreSQL databases:

   ```yaml
   postgres:
     useDefault: false
     host: HOST_ADDRESS
     port: PORT_NUMBER
     user: YOUR_USER_NAME
     database: YOUR_DATABASE
     passwordSecret: secret-name
     sslMode: require
   ```

   To create the `passwordSecret`, run `kubectl create secret generic
   secret-name --from-literal=password=UserDefinedPassword` (be sure to replace
   `UserDefinedPassword` with your actual password).

   You can find/define these values in your [PostgreSQL server configuration
   file](https://www.postgresql.org/docs/current/config-setting.html).

5. [Enable Dev URL Usage](../admin/devurls.md). Dev URLs allow users to access
   the web servers running in your environment. To enable, provide a wildcard
   domain and its DNS certificate and update your helm chart accordingly. This
   step is **optional** but recommended.

6. After you've created the pod, tail the logs to find the randomly generated
   password for the admin user

   ```bash
   kubectl logs -n coder -l coder.deployment=cemanager -c cemanager \
    --tail=-1 | grep -A1 -B2 Password
   ```

   When this step is done, you will see:

   ```text
   ----------------------
   User:     admin
   Password: kv...k3
   ----------------------
   ```

   These are the credentials you need to continue setup using Coder's web UI.

> If you lose your admin credentials, you can use the [admin password
> reset](../admin/access-control/password-reset.md#resetting-the-site-admin-password)
> process to regain access.

## Accessing Coder

1. To access Coder's web UI, you'll need to get its IP address by running the
   following in the terminal to list the Kubernetes services running:

   ```bash
   kubectl --namespace coder get services
   ```

   The row for the **ingress-nginx** service includes an **EXTERNAL-IP** value;
   this is the IP address you need.

2. In your browser, navigate to the external IP of ingress-nginx.

3. Use the admin credentials you obtained in this installation guide's previous
   step to log in to the Coder platform. If this is the first time you've logged
   in, Coder will prompt you to change your password.

At this point, you're ready to proceed to [configuring Coder](configuration.md).
