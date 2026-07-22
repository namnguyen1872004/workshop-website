---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---


# Introducing the LBC Ingress-to-Gateway API migration toolkit

While learning about Amazon EKS and Kubernetes traffic management on AWS, I found an AWS Blog article introducing the **LBC Ingress-to-Gateway API Migration Toolkit**.

Ingress has traditionally been used to route external traffic to Services running in Kubernetes. On AWS, the AWS Load Balancer Controller can read Ingress resources and automatically provision an Application Load Balancer, listeners, target groups, and routing rules.

However, as Kubernetes environments become more complex, Ingress configurations can become difficult to maintain. Many advanced capabilities depend on controller-specific annotations, which can make YAML manifests long, difficult to validate, and tightly coupled to a specific controller.

Gateway API addresses these limitations by introducing a more structured, extensible, and role-oriented traffic-management model.

Link blog: https://www.facebook.com/groups/awsstudygroupfcj/posts/2220037625427864

## Why Migrate to Gateway API?

Gateway API is designed as the long-term Kubernetes interface for managing application traffic.

Instead of placing most routing configuration in a single Ingress resource, Gateway API separates responsibilities across several resource types:

- `GatewayClass` identifies the controller responsible for a Gateway.
- `Gateway` defines traffic entry points, listeners, and protocols.
- `HTTPRoute` defines hostnames, paths, backends, and routing rules.
- `ReferenceGrant` controls cross-namespace references.
- Policy resources extend behavior for specific infrastructure requirements.

This model creates a clearer separation between infrastructure administrators and application developers.

Infrastructure teams can manage GatewayClass, Gateway, listeners, and load-balancer settings. Application teams can independently manage HTTPRoute resources for their services.

Gateway API also provides capabilities such as:

- Weighted traffic splitting.
- Canary deployments.
- Cross-namespace routing.
- Admission-time schema validation.
- Typed and extensible resources.
- Better support for multi-team Kubernetes environments.

## Challenges of Manual Migration

Migrating from LBC Ingress to Gateway API involves more than changing `kind: Ingress` to `kind: Gateway`.

A production Ingress can contain annotations for:

- Load balancer scheme.
- Target type.
- HTTP and HTTPS listeners.
- AWS Certificate Manager certificates.
- HTTP-to-HTTPS redirects.
- Health-check paths.
- Host-based and path-based routing.
- URI rewrites.
- IngressGroup configuration.
- Target-group behavior.

During a manual migration, administrators must determine how each annotation maps to Gateway API and AWS Load Balancer Controller Gateway resources.

For example:

- `ingressClassName: alb` becomes a GatewayClass.
- Scheme and certificate settings become LoadBalancerConfiguration.
- HTTP and HTTPS ports become Gateway listeners.
- SSL redirects become an HTTPRoute with a RequestRedirect filter.
- Host, path, and Service rules become HTTPRoute resources.
- Health checks and target types become TargetGroupConfiguration.

A mistake in a listener, certificate, backend, or routing rule can disrupt production traffic. The Migration Toolkit reduces this risk by automating the translation and providing a validation process before traffic is changed.

## Migration Toolkit Components

The toolkit contains two complementary components.

### 1. lbc-migrate CLI

`lbc-migrate` is a command-line tool that reads LBC Ingress resources and generates equivalent Gateway API manifests.

It can translate:

- AWS Load Balancer Controller annotations.
- Host and path rules.
- TLS termination.
- SSL redirects.
- IngressGroup configuration.
- Load balancer settings.
- Target-group settings.

The tool can read static YAML files or inspect resources directly from a running Kubernetes cluster.

Existing Deployments and Services are reused. The generated resources focus on the routing layer, including GatewayClass, Gateway, HTTPRoute, LoadBalancerConfiguration, and TargetGroupConfiguration.

### 2. Migration Console

Migration Console is a local web interface bundled with the `lbc-migrate` binary.

It provides a read-only review experience where administrators can:

- Inspect existing Ingress resources.
- Review generated Gateway API manifests.
- Compare Ingress and Gateway plans.
- Examine dry-run results.
- Identify annotations requiring manual changes.

The console helps users review the migration without directly modifying production resources.

## Prerequisites

Before starting the migration, prepare:

- A running Amazon EKS cluster.
- A managed node group or suitable worker nodes.
- A supported AWS Load Balancer Controller version.
- Gateway API standard CRDs.
- AWS Load Balancer Controller Gateway CRDs.
- An application currently exposed through an ALB Ingress.
- An AWS Certificate Manager certificate when HTTPS is used.
- Kubernetes permissions to create and inspect the required resources.

Install the standard Gateway API CRDs:

```bash
kubectl apply --server-side=true \
  -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.5.0/standard-install.yaml
```

Install the LBC Gateway CRDs:

```bash
kubectl apply \
  -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.4.0/config/crd/gateway/gateway-crds.yaml
```

Confirm that the controller recognizes the Gateway API resources:

```bash
kubectl -n kube-system logs deploy/aws-load-balancer-controller \
  | grep "gateway.k8s.aws/alb"
```

## Migration Workflow

The migration can be completed in six controlled steps.

### Step 1 – Install lbc-migrate

Clone the AWS Load Balancer Controller repository and build the migration binary:

```bash
git clone https://github.com/kubernetes-sigs/aws-load-balancer-controller.git
cd aws-load-balancer-controller

make lbc-migrate
make install-lbc-migrate
```

After installation, the `lbc-migrate` command can be used from the terminal.

### Step 2 – Translate Ingress Resources

Read the configuration directly from a live cluster:

```bash
lbc-migrate \
  --from-cluster \
  --namespaces demo \
  --output-dir ./gateway-output
```

The tool reads related resources, including:

- Ingress.
- Service.
- IngressClass.
- IngressClassParams.

It then creates Gateway API YAML manifests.

The generated output can include:

- GatewayClass.
- Gateway.
- HTTPRoute for HTTPS redirects.
- HTTPRoute for application routing.
- LoadBalancerConfiguration.
- TargetGroupConfiguration.

If an annotation cannot be translated safely, the tool produces a warning rather than silently generating an incorrect configuration.

## Step 3 – Preview with Dry Run

By default, generated Gateway resources include the following annotation:

```yaml
gateway.k8s.aws/dry-run: "true"
```

Apply the manifests:

```bash
kubectl apply -f ./gateway-output/gateway-resources.yaml
```

In dry-run mode, the controller validates and models the required AWS resources without provisioning a real Application Load Balancer.

This allows administrators to review:

- Listeners.
- Routing rules.
- Certificates.
- Target groups.
- Health checks.
- Annotation mappings.
- Configuration requiring manual changes.

Dry-run validation should be completed before production resources are created.

## Step 4 – Complete Unsupported Transformations

Not every Ingress annotation can be translated automatically.

One example is a URI rewrite that uses a regular expression and capture group.

An Ingress can remove the `/api` prefix before forwarding a request to the backend. Gateway API uses a structured URLRewrite filter, so a dynamic regular-expression replacement may require a manual change.

The HTTPRoute can be updated as follows:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: demo-route
  namespace: demo
spec:
  hostnames:
    - demo.example.com
  parentRefs:
    - name: demo-gateway
      sectionName: https-443
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      filters:
        - type: URLRewrite
          urlRewrite:
            path:
              type: ReplacePrefixMatch
              replacePrefixMatch: /
      backendRefs:
        - name: echoserver
          port: 80
```

This converts:

```text
/api/hello
```

into:

```text
/hello
```

before the request reaches the backend Service.

## Step 5 – Deploy and Verify the Gateway

After reviewing the dry-run output, regenerate the manifests with dry run disabled:

```bash
lbc-migrate \
  --from-cluster \
  --namespaces demo \
  --output-dir ./gateway-live \
  --dry-run=false
```

Apply the resources:

```bash
kubectl apply -f ./gateway-live/gateway-resources.yaml
```

The AWS Load Balancer Controller creates a new ALB for the Gateway API resources. The original Ingress ALB continues serving traffic.

Check the Gateway status:

```bash
kubectl get gateway demo-gateway -n demo
```

The Gateway should report:

```text
PROGRAMMED: True
```

Verify:

- Target groups are healthy.
- HTTP redirects to HTTPS correctly.
- The TLS certificate is valid.
- Host and path routing behave correctly.
- URI rewrites match the original behavior.
- Amazon CloudWatch metrics show no unexpected errors.

## Step 6 – Shift Traffic and Remove the Old Ingress

After the Gateway ALB has been validated, traffic can be gradually shifted from the Ingress ALB to the Gateway ALB.

Possible approaches include:

- Amazon Route 53 weighted routing.
- AWS Global Accelerator.
- Another traffic-management mechanism supported by the environment.

A gradual cutover allows the new path to be monitored before all users are migrated.

If a problem occurs, traffic can be shifted back to the Ingress ALB.

When the Gateway path has operated successfully for a sufficient period, remove the old Ingress:

```bash
kubectl delete ingress demo -n demo
```

The AWS Load Balancer Controller automatically removes the old ALB, listener rules, and target groups associated with the Ingress.

## Migration Recommendations

Important recommendations include:

- Always start with a dry run.
- Review annotation compatibility before migration.
- Do not delete the existing Ingress immediately.
- Keep the Ingress and Gateway ALBs running in parallel during validation.
- Shift traffic gradually.
- Monitor target health and Amazon CloudWatch metrics.
- Prepare a rollback procedure.
- Review URI rewrites, authentication, and custom annotations carefully.
- For new workloads, consider using HTTPRoute from the beginning.

## Benefits of the Migration Toolkit

The toolkit provides several benefits:

- Reduces the time required to rewrite manifests.
- Limits annotation-mapping mistakes.
- Generates structured Gateway API resources.
- Provides dry-run validation before AWS resources are created.
- Allows Ingress and Gateway resources to run simultaneously.
- Supports controlled traffic migration.
- Reuses existing Deployments and Services.
- Warns about configurations that require manual changes.
- Supports a phased migration strategy.

## Conclusion

This article helped me understand how to migrate AWS Load Balancer Controller Ingress resources to Gateway API on Amazon EKS.

Gateway API is more than a new YAML format. It introduces a clearer traffic-management model with better separation of responsibilities, extensibility, validation, and support for modern routing strategies.

The `lbc-migrate` CLI automates most of the resource translation, while Migration Console and dry-run validation allow administrators to review the result before production changes are made.

In my opinion, the most valuable part of this approach is that the new Gateway ALB can run alongside the existing Ingress ALB. This makes it possible to validate the new architecture, shift traffic gradually, and roll back when necessary.

The toolkit is therefore useful for Amazon EKS environments that currently use LBC Ingress and plan to adopt Gateway API as their long-term Kubernetes routing interface.

<p align="center">
  <strong>Reference Article:</strong>
  <a href="https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-the-lbc-ingress-to-gateway-api-migration-toolkit/" target="_blank">
    AWS Networking & Content Delivery Blog
  </a>
</p>