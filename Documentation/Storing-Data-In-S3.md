# Storing Data in S3

Chargeback supports storing collected usage information and generated reports in
S3. Data stored locally in the cluster (which is the default) will not survive
restarts of the hive pod. By configuring chargeback to store data in S3, this
data will become persistent.

## Set AWS Credentials

The install script will detect if AWS credentials are stored in the environment
variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, and ask if these
credentials should be used to create a secret in Kubernetes for Chargeback to
use. This is required to access S3 buckets.

## Set AWS region

Chargeback currently need to be configured to use a specific AWS region. This is
only temporary, and this requirement should be removed in the future. Until
then, change the `aws-region` value in
`manifests/chargeback/chargeback-config.yaml`.

## Modify Chargeback data stores

Chargeback has a few CRDs that are installed to control what types of reports
can be generated. More information on this is available in the [documentation on
Chargeback's CRD model][crd-model], but in summary the data store CRDs describe
where data for a given Prometheus query should be stored. These data stores can
be modified to point to a S3 bucket instead of local storage.

The default data stores shipped with chargeback are located in
`manifests/custom-resources/datastores`. For each data store with a `promsum`
section, replace:

```
storage:
  local: {}
```

with:

```
storage:
  s3:
    bucket: MY-BUCKET-NAME
    prefix: MY-PREFIX
```

As an example, here's the `pod-cpu-usage.yaml` file after being modified to
store data in the `chargeback` bucket under the `promsum/cpu_by_pod` prefix:

```
apiVersion: chargeback.coreos.com/v1alpha1
kind: ReportDataStore
metadata:
  name: "pod-cpu-usage"
  labels:
    tectonic-chargeback: "true"
spec:
  promsum:
    queries:
    - "get-cpu-by-pod"
  storage:
    s3:
      bucket: chargeback
      prefix: promsum/cpu_by_pod
```

## Set an output location on reports

Reports also must specify to put report results in S3 if they shouldn't be
stored locally in the cluster. To alter the example reports in
`manifests/custom-resources/reports` to store data in S3, replace:

```
output:
  local: {}
```

with:

```
output:
  s3:
    bucket: MY-BUCKET-NAME
    prefix: MY-PREFIX
```

[crd-model]: CRD-Model.md