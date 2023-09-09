# keycloak

Keycloak deployment in internal k8s instance.

The keycloak configuration is stored in this repository (in `realms.yaml`) as an encrypted config map. It is applied
automatically during a helm upgrade as a post-install and post-upgrade hook.

In order to update the configuration without performing an upgrade:

1. decrypt the `realms.yaml` file:

    ```
    sops --decrypt --in-place keycloak/realms.yaml
    ```

2. make the necessary modifications to the config
3. re-encrypt the `realms.yaml` in place:

    ```
    sops --encrypt --in-place keycloak/realms.yaml
    ```

4. modify the `release.yaml` file, bumping the value of the `spec.values.keycloakConfigCli.annotations.nise/executions`
   annotation such that the hook will be reconciled and re-executed
5. commit and push the changes to git
6. wait for the Flux reconciliation or trigger it directly with

    ```
    flux reconcile source git [git repo name] -n [namespace] && flux reconcile kustomization [kustomization name] -n [namespace] --verbose
    ```