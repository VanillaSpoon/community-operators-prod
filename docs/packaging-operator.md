## Package your Operator

This repository makes use of the [Operator Framework](https://github.com/operator-framework) and its packaging concept for Operators. Your contribution is welcome in the form of a _Pull Request_ with your Operator packaged for use with [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager/).

### Packaging format

Your Operator submission can be formatted following the `bundle` or `packagemanifest` format. The `packagemanifest` format is a legacy format that is kept for backward compatibility only and then, it strongly recommended to use `bundle` format. The former allows shipping your entire Operator with all its versions in one single directory. The latter allows shipping individual releases in container images.

In general, a released version of your Operator is described in a `ClusterServiceVersion` manifest alongside the `CustomResourceDefinitions` of your Operator and additional metadata describing your Operator listing.

#### Create a ClusterServiceVersion

To add your operator to any of the supported platforms, you will need to submit metadata for your Operator to be used by the [Operator Lifecycle Manager](https://github.com/operator-framework/operator-lifecycle-manager/) (OLM). This is a YAML file called `ClusterServiceVersion` which contains references to all of the CRDs, RBAC rules, `Deployment` and container image needed to install and securely run your Operator. It also contains user-visible info like a description of its features and supported Kubernetes versions. Note that your Operator's CRDs are shipped in separate manifests alongside the CSV so OLM can register them during installation (your Operator is not supposed to self-register its CRDs).

Follow [this guide](https://olm.operatorframework.io/docs/tasks/creating-operator-manifests/) to create an OLM-compatible CSV for your operator. You can also leverage existing examples in this repository.

For more information about the advanced features of the Operator metadata format used here, be sure to check the [Operator Lifecycle Manager documentation](https://olm.operatorframework.io/docs/advanced-tasks/) about how to package webhooks, upgrade readiness probes or an Operator supported on multiple computer architectures (multi-arch).

##### Categories #####

An Operator's CSV must contain the fields mentioned [here](./packaging-required-fields.md#packaging-required-fields-for-operatorhub) for it to be displayed properly within the various platforms. If your operator needs a new category, follow the instructions about [categories](./packaging-required-fields.md#categories).

There is one CSV per version of your Operator alongside the CRDs.

#### Create a release

The `bundle` format has a top-level directory named after your Operator name in the `ClusterServiceVersion` directory. Inside are sub-directories for the individual bundle, named after the [semantic versioning](https://semver.org) release of your Operator.

All metadata is defined within the individual release of the Operator. That is, inside each bundle. This includes the channel definitions. The default channel is also defined within the bundle and overwritten by every new bundle you add (this is a known limitation and is being worked on).

Within each version, you have your `CustomResourceDefinitions`, `ClusterServiceVersion` file (containing the same name and version of your Operator as defined inside the YAML structure) and some metadata about the bundle. You can [learn more about the bundle format here](https://olm.operatorframework.io/docs/tasks/creating-operator-bundle/) and also see an [example](https://github.com/operator-framework/bundle-example).

Note that this community project only requires you to submit your bundle in the form of metadata. The integrated release pipeline of this repository will take care of publishing your bundle as a container image and maintaining it in a public catalog.

Your directory structure might look like this when using the `bundle` format. Notice that the `Dockerfile` is optionally and actually ignored. The processing pipeline of this site builds a container image for each of your bundles regardless.

```console
$ tree my-operator/

my-operator/
├── 0.1.0
│   ├── manifests
│   │   ├── my-operator-crd1.crd.yaml
│   │   ├── my-operator-crd2.crd.yaml
│   │   ├── my-operator-crd3.crd.yaml
│   │   └── my-operator.v0.1.0.clusterserviceversion.yaml
│   ├── metadata
│   │   └── annotations.yaml
│   └── Dockerfile
├── 0.5.0
│   ├── manifests
│   │   ├── my-operator-crd1.crd.yaml
│   │   ├── my-operator-crd2.crd.yaml
│   │   ├── my-operator-crd3.crd.yaml
│   │   └── my-operator.v0.5.0.clusterserviceversion.yaml
│   ├── metadata
│   │   └── annotations.yaml
│   └── Dockerfile
├── 1.0.0
│   ├── manifests
│   │   ├── my-operator-crd1.crd.yaml
│   │   ├── my-operator-crd2.crd.yaml
│   │   ├── my-operator-crd3.crd.yaml
│   │   └── my-operator.v1.0.0.clusterserviceversion.yaml
│   ├── metadata
│   │   └── annotations.yaml
│   └── Dockerfile
└── 2.0.0
    ├── manifests
    │   ├── my-operator-crd1.crd.yaml
    │   ├── my-operator-crd2.crd.yaml
    │   ├── my-operator-crd3.crd.yaml
    │   └── my-operator.v2.0.0.clusterserviceversion.yaml
    ├── metadata
    │   └── annotations.yaml
    └── Dockerfile
...
```
If you used `operator-sdk` to develop your Operator you can also leverage its packaging tooling to [create a bundle](https://sdk.operatorframework.io/docs/olm-integration/quickstart-bundle/#creating-a-bundle) by just running the target `make bundle`.

#### Moving from `packagemanifest` to `bundle` format

Eventually, this repository will only accept bundle format at some point in the future. Also, the `bundle` format has more features like `semver` mode or, in the future, installing bundles directly outside of a catalog.

Migration of existing content, regardless of whether the Operator was created with the SDK, can be achieved with the `opm` tool on a per Operator version basis. You can download `opm` [here](https://github.com/operator-framework/operator-registry/releases/latest).

Suppose `v2.0.0` is the version of the Operator you want to test convert to bundle format directory with the `opm` tool:

```console
mkdir /tmp/my-operator-2.0.0-bundle/
cd /tmp/my-operator-2.0.0-bundle/
opm alpha bundle build --directory /path/to/my-operator/2.0.0-bundle/ --tag my-operator-bundle:v2.0.0 --output-dir .
```

This will have generated the bundle format layout in the current working directory `/tmp/my-operator-2.0.0-bundle/`:

```console
$ tree .

/tmp/my-operator-2.0.0-bundle/
├── manifests
│   ├── my-operator-crd1.crd.yaml
│   ├── my-operator-crd2.crd.yaml
│   ├── my-operator-crd3.crd.yaml
│   └── my-operator.v2.0.0.clusterserviceversion.yaml
├── metadata
│   └── annotations.yaml
└── bundle.Dockerfile
```

You can verify the generated bundle metadata for semantic correctness with the `operator-sdk` on this directory.

```console
operator-sdk bundle validate /tmp/my-operator-2.0.0-bundle/ --select-optional name=operatorhub
```

#### About the Dockerfile

A `Dockerfile` is typically part of the bundle metadata used to build the bundle image. For security reasons, our release process is generating an internal `Dockerfile` that is used to build and publish the bundle image. Existing `Dockerfile` or `bundle.Dockerfile` will be ignored.  You can leverage the `annotations.yaml` file to control custom labels the resulting image should have. For example:

```
annotations:
  # Core bundle annotations.
  operators.operatorframework.io.bundle.mediatype.v1: registry+v1
  operators.operatorframework.io.bundle.manifests.v1: manifests/
  operators.operatorframework.io.bundle.metadata.v1: metadata/
  operators.operatorframework.io.bundle.package.v1: global-load-balancer-operator
  operators.operatorframework.io.bundle.channels.v1: alpha
  operators.operatorframework.io.bundle.channel.default.v1: alpha
  operators.operatorframework.io.metrics.mediatype.v1: metrics+v1
  operators.operatorframework.io.metrics.builder: operator-sdk-v1.4.0+git
  operators.operatorframework.io.metrics.project_layout: go.kubebuilder.io/v3

  # Annotations for testing.
  operators.operatorframework.io.test.mediatype.v1: scorecard+v1
  operators.operatorframework.io.test.config.v1: tests/scorecard/

```
will generate `Dockerfile`

```
FROM scratch

# from metadata/annotations.yaml
LABEL operators.operatorframework.io.bundle.mediatype.v1="registry+v1"
LABEL operators.operatorframework.io.bundle.manifests.v1="manifests/"
LABEL operators.operatorframework.io.bundle.metadata.v1="metadata/"
LABEL operators.operatorframework.io.bundle.package.v1="global-load-balancer-operator"
LABEL operators.operatorframework.io.bundle.channels.v1="alpha"
LABEL operators.operatorframework.io.bundle.channel.default.v1="alpha"
LABEL operators.operatorframework.io.metrics.mediatype.v1="metrics+v1"
LABEL operators.operatorframework.io.metrics.builder="operator-sdk-v1.4.0+git"
LABEL operators.operatorframework.io.metrics.project_layout="go.kubebuilder.io/v3"
LABEL operators.operatorframework.io.test.mediatype.v1="scorecard+v1"
LABEL operators.operatorframework.io.test.config.v1="tests/scorecard/"

COPY ./manifests manifests/
COPY ./metadata metadata/
COPY ./tests/scorecard/ tests/scorecard/
```
!!! note
    If you specify the `operators.operatorframework.io.test.config.v1` to embed [scorecard](https://sdk.operatorframework.io/docs/advanced-topics/scorecard/scorecard/) tests in your bundle, make sure the supplied directory path (e.g. `tests/scorecard/` relative from the bundle root directory) exists, otherwise, the validation will fail.

You can download `operator-sdk` [here](https://github.com/operator-framework/operator-sdk/releases/latest).

#### Operator icon ####
The icon is defined in a CSV as `spec.icon`. If you don't have an own icon, you should use a default one:
```
  icon:
    - base64data: "PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNTguNTEgMjU4LjUxIj48ZGVmcz48c3R5bGU+LmNscy0xe2ZpbGw6I2QxZDFkMTt9LmNscy0ye2ZpbGw6IzhkOGQ4Zjt9PC9zdHlsZT48L2RlZnM+PHRpdGxlPkFzc2V0IDQ8L3RpdGxlPjxnIGlkPSJMYXllcl8yIiBkYXRhLW5hbWU9IkxheWVyIDIiPjxnIGlkPSJMYXllcl8xLTIiIGRhdGEtbmFtZT0iTGF5ZXIgMSI+PHBhdGggY2xhc3M9ImNscy0xIiBkPSJNMTI5LjI1LDIwQTEwOS4xLDEwOS4xLDAsMCwxLDIwNi40LDIwNi40LDEwOS4xLDEwOS4xLDAsMSwxLDUyLjExLDUyLjExLDEwOC40NSwxMDguNDUsMCwwLDEsMTI5LjI1LDIwbTAtMjBoMEM1OC4xNiwwLDAsNTguMTYsMCwxMjkuMjVIMGMwLDcxLjA5LDU4LjE2LDEyOS4yNiwxMjkuMjUsMTI5LjI2aDBjNzEuMDksMCwxMjkuMjYtNTguMTcsMTI5LjI2LTEyOS4yNmgwQzI1OC41MSw1OC4xNiwyMDAuMzQsMCwxMjkuMjUsMFoiLz48cGF0aCBjbGFzcz0iY2xzLTIiIGQ9Ik0xNzcuNTQsMTAzLjQxSDE0MS42NkwxNTQuOSw2NS43NmMxLjI1LTQuNC0yLjMzLTguNzYtNy4yMS04Ljc2SDEwMi45M2E3LjMyLDcuMzIsMCwwLDAtNy40LDZsLTEwLDY5LjYxYy0uNTksNC4xNywyLjg5LDcuODksNy40LDcuODloMzYuOUwxMTUuNTUsMTk3Yy0xLjEyLDQuNDEsMi40OCw4LjU1LDcuMjQsOC41NWE3LjU4LDcuNTgsMCwwLDAsNi40Ny0zLjQ4TDE4NCwxMTMuODVDMTg2Ljg2LDEwOS4yNCwxODMuMjksMTAzLjQxLDE3Ny41NCwxMDMuNDFaIi8+PC9nPjwvZz48L3N2Zz4="
      mediatype: "image/svg+xml"
```
Supported formats: svg, jpg, png

### Updating your existing Operator

Unless of a purely cosmetic nature, subsequent updates to your Operator should result in new `bundle` directories being added, containing an updated CSV as well as copied, updated and/or potentially newly added CRDs. Within your new CSV, update the `spec.version` field to the desired new semantic version of your Operator.

In order to have OLM enable updates to your a new Operator version you can choose between three update modes: `semver-mode`, `semver-skippatch-mode` and `replaces-mode`. The default is `semver-mode`. If you want to change the default, place a file called `ci.yaml` in your top-level directory (works for both `packagemanifest` or `bundle` format) and set it to either of the two other values. For example:

```yaml
updateGraph: replaces-mode
```

#### semver-mode
OLM treats all your Operator versions with semantic version rules and updates them in order of those versions. That is, every version will be replaced by the next higher version according to a semantic versioning sort order. During an update on the cluster, OLM will update to the latest version, one version at a time. To use this, simply specify `spec.version` in your CSV. If you accidentally add `spec.replaces` this will contradict semantic versioning and raise an error.

#### semver-skippatch
Works like `semver` with slightly different behavior of OLM on the cluster, where instead of updating from e.g. `1.1.0` and an update path according to a semver ordering rules like so: `1.1.0 -> 1.1.1 -> 1.1.2`, the update would jump straight to `1.1.2` instead of updating to `1.1.1` first.

#### replaces-mode
Each Operator bundle not only contains `spec.version` but also points to an older version it can upgrade from via `spec.replaces` key in the CSV file, e.g. `replaces: my-operator.v1.0.0`. From this chain of back pointers, OLM computes the *update graph* at runtime. This allows us to omit some versions from the *update graph* or release special leaf versions.

Regardless of which mode you choose to have OLM create update paths for your Operator, it continuously update your Operator often as new features are added and bugs are fixed.

##### Channels #####

Use channels to allow your users to select a different update cadence, e.g. `stable` vs. `nightly`. If you have only a single channel the use of `defaultChannel` is optional.

An example of `my-operator.package.yaml`:

```yaml
packageName: my-operator
channels:
- name: stable
  currentCSV: my-operator.v1.0.0
- name: nightly
  currentCSV: my-operator.v1.0.3-beta
defaultChannel: stable
```

Your CSV versioning should follow [semantic versioning](https://semver.org/) concepts. Again, `packageName` is the suffix of the `package.yaml` file name and the field in `spec.name` in the CSV should all refer to the same Operator name.

### Operator Bundle Editor

You can now create your Operator bundle using the [bundle editor](https://operatorhub.io/packages). Starting by uploading your Kubernetes YAML manifests, the forms on the page will be populated with all valid information and used to create the new Operator bundle. You can modify or add properties through these forms as well. The result will be a downloadable ZIP file.

## Provide information about your Operator

A large part of the information gathered in the CSV is used for user-friendly visualization on [OperatorHub.io](https://operatorhub.io) or components like the embedded OperatorHub in OpenShift. Your work is on display, so please ensure to provide relevant information in your Operator's description, specifically covering:

* What the managed application is about and where to find more information
* The features of your Operator and how to use it
* Any manual steps required to fulfill pre-requisites for running/installing your Operator
