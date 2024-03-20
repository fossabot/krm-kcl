# Kubernetes Resource Model (KRM) KCL Specification

[![Go Report Card](https://goreportcard.com/badge/kcl-lang.io/krm-kcl)](https://goreportcard.com/report/kcl-lang.io/krm-kcl)
[![GoDoc](https://godoc.org/kcl-lang.io/krm-kcl?status.svg)](https://godoc.org/kcl-lang.io/krm-kcl)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://kcl-lang.io/krm-kcl/blob/main/LICENSE)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fkcl-lang%2Fkrm-kcl.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fkcl-lang%2Fkrm-kcl?ref=badge_shield)

The KRM KCL function SDK contains a KRM KCL spec and an interpreter to run KCL codes to generate, mutate or validate Kubernetes resources.

The KCL programming language can be used to:

+ Add labels or annotations based on a condition.
+ Inject a sidecar container in all KRM resources that contain a `PodTemplate`.
+ Validate all KRM resources using KCL schema.
+ Use an abstract model to generate KRM resources.

## Test the KRM Function

### Unit Tests

You need to put your KCL script source in the functionConfig of kind KCLRun and then the function will run the KCL script that you provide.

This function can be used both declaratively and imperatively.

```bash
make test
```

### Integration Tests

You need to put your KCL source code or url in the functionConfig of kind KCLRun and then the function will run the KCL code that you provide.

```bash
# Verify that the annotation is added to the `Deployment` resource and the other resource `Service` 
# does not have this annotation.
export TEST_FILE=./pkg/options/testdata/yaml_stream/kcl-run-code.yaml
diff \
  <(cat ${TEST_FILE}) \
  <(cat ${TEST_FILE} | go run main.go)
```

## FunctionConfig

To use a `KCLRun` as the functionConfig, the KCL source must be specified in the source field. Additional parameters can be specified in the params field. The params field supports any complex data structure as long as it can be represented in YAML.

```yaml
apiVersion: krm.kcl.dev/v1alpha1
kind: KCLRun
metadata:
  name: conditionally-add-annotations
spec:
  params:
    toMatch:
      config.kubernetes.io/local-config: "true"
    toAdd:
      configmanagement.gke.io/managed: disabled
  source: |
    params = option("params")
    toMatch = params.toMatch
    toAdd = params.toAdd
    items = [item | {
       # If all annotations are matched, patch more annotations
       if all key, value in toMatch {
          item.metadata.annotations[key] == value
       }:
           metadata.annotations: toAdd
    } for item in option("items")]
```

In the example above, the script accesses the `toMatch` parameters using `option("params").toMatch`.

Besides, the `source` field supports different KCL sources, which can come from a local file, VCS such as github, OCI registry, http, etc. You can see the specific usage [here](./pkg/options/testdata/). Take an OCI source as the example.

```yaml
apiVersion: krm.kcl.dev/v1alpha1
kind: KCLRun
spec:
  params:
    annotations:
      config.kubernetes.io/local-config: "true"
  source: oci://ghcr.io/kcl-lang/set-annotation
```

## Guides for Developing KCL

Here's what you can do in the KCL script:

+ Read resources from `option("resource_list")`. The `option("resource_list")` complies with the [KRM Functions Specification](https://github.com/kubernetes-sigs/kustomize/blob/master/cmd/config/docs/api-conventions/functions-spec.md#krm-functions-specification). You can read the input resources from `option("items")` and the `functionConfig` from `option("functionConfig")`.
+ Return a KRM list for output resources.
+ Return an error using `assert {condition}, {error_message}`.
+ Read the environment variables. e.g. `option("PATH")`.
+ Read the OpenAPI schema. e.g. `option("open_api")["definitions"]["io.k8s.api.apps.v1.Deployment"]` (**Not yet implemented**).

## Library

You can directly use [KCL standard libraries](https://kcl-lang.io/docs/reference/model/overview) without importing them, such as `regex.match`, `math.log`.

## Tutorial

+ See [here](https://kcl-lang.io/docs/reference/lang/tour) to study more features of KCL.

## Examples

+ See [here](./examples/README.md) for more examples.

## Tools and Integrations

+ [Kubectl KCL Plugin](https://github.com/kcl-lang/kubectl-kcl)
+ [Helm KCL Plugin](https://github.com/kcl-lang/helm-kcl)
+ [Helmfile KCL Plugin](https://github.com/kcl-lang/helmfile-kcl)
+ [KPT KCL Plugin](https://github.com/kcl-lang/kpt-kcl)
+ [KCL Operator](https://github.com/kcl-lang/kcl-operator)
+ [Crossplane KCL Function](https://github.com/crossplane-contrib/function-kcl)


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fkcl-lang%2Fkrm-kcl.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fkcl-lang%2Fkrm-kcl?ref=badge_large)