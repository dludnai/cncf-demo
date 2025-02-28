# Managing Secrets From Azure With External Secrets Operator (ESO)

TODO: Intro

## Setup

```bash
export RESOURCE_GROUP=$(\
    yq ".production.azure.resourceGroup" settings.yaml)

az keyvault create --name cncf-demo \
    --resource-group $RESOURCE_GROUP --location eastus \
    | tee azure-keyvault.json

az keyvault secret set --name production-postgresql \
    --vault-name cncf-demo \
    --value '{"password": "YouWillNeverFindOut"}'

export CLIENT_ID=$(jq --raw-output ".clientId" azure-creds.json)

export CLIENT_SECRET=$(\
    jq --raw-output ".clientSecret" azure-creds.json)

export TENANT_ID=$(jq --raw-output ".tenantId" azure-creds.json)

kubectl --namespace external-secrets \
    create secret generic azure \
    --from-literal ClientID=$CLIENT_ID \
    --from-literal ClientSecret=$CLIENT_SECRET

yq --inplace ".spec.provider.azurekv.tenantId = \"$TENANT_ID\"" \
    eso/secret-store-azure.yaml

cat eso/secret-store-azure.yaml

cp eso/secret-store-azure.yaml infra/.

git add .

git commit -m "Secret Store"

git push

kubectl --namespace external-secrets get clustersecretstores

# TODO: Fix the complaint about permissions

# Wait until the ClusterSecretStore appears
```

## How Did You Define Your App?

* [Helm](helm.md)
* [Kustomize](kustomize.md)
* [Carvel ytt](carvel.md)
* [cdk8s](cdk8s.md)
