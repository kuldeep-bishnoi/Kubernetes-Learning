# Kubernetes Security

This directory contains information about Kubernetes security concepts, features, and best practices.

## Introduction

Security is a critical aspect of Kubernetes deployments, especially in production environments. Kubernetes provides multiple layers of security that can be configured to protect your applications, data, and infrastructure.

## Table of Contents

1. [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
2. [Pod Security Standards](#pod-security-standards)
3. [Network Policies](#network-policies)
4. [Secrets Management](#secrets-management)
5. [Security Contexts](#security-contexts)
6. [Service Accounts](#service-accounts)
7. [Authentication and Authorization](#authentication-and-authorization)
8. [Admission Controllers](#admission-controllers)
9. [Image Security](#image-security)
10. [Runtime Security](#runtime-security)
11. [Auditing](#auditing)
12. [Examples](#examples)
13. [Best Practices](#best-practices)
14. [Further Reading](#further-reading)

## Role-Based Access Control (RBAC)

RBAC is a method of regulating access to Kubernetes resources based on the roles of individual users within your organization. It uses several Kubernetes objects:

- **Roles** and **ClusterRoles**: Define permissions to access resources
- **RoleBindings** and **ClusterRoleBindings**: Bind roles to users, groups, or service accounts

RBAC allows you to control who can access what resources and what actions they can perform, adhering to the principle of least privilege.

## Pod Security Standards

Pod Security Standards (PSS) replaced the deprecated Pod Security Policies (PSPs) in Kubernetes 1.25. They define different levels of security for pods:

- **Privileged**: Unrestricted policies with access to all features
- **Baseline**: Minimally restrictive policies that prevent known privilege escalations
- **Restricted**: Heavily restricted policies, following security best practices

These standards can be enforced using admission controllers and namespace labels.

## Network Policies

Network Policies are like firewall rules for pods. They allow you to:

- Control which pods can communicate with each other
- Define egress and ingress rules
- Create network segmentation for multi-tenant environments
- Implement a zero-trust networking model

## Secrets Management

Managing sensitive information in Kubernetes:

- Understanding Kubernetes Secrets
- External secrets management solutions (Vault, CSI drivers)
- Secret encryption at rest
- Secret rotation and lifecycle management

## Security Contexts

Security Contexts define privilege and access control settings for Pods and Containers, including:

- User and group IDs
- Privilege mode settings
- Linux capabilities
- SELinux context
- AppArmor and Seccomp profiles

## Service Accounts

Service Accounts provide identities for processes running in a Pod, allowing:

- Authentication to the Kubernetes API
- RBAC permissions for pods
- Token mounting and management
- Integration with external identity providers

## Authentication and Authorization

Understanding how Kubernetes handles:

- User authentication methods
- Integration with external identity providers (OIDC, LDAP)
- Authorization mechanisms
- API request flow

## Admission Controllers

Admission controllers are plugins that govern and enforce policies on requests to the Kubernetes API:

- Validating admission controllers
- Mutating admission controllers
- Policy engines (OPA/Gatekeeper, Kyverno)

## Image Security

Ensuring the security of container images:

- Image scanning for vulnerabilities
- Signed and verified images
- Private registries
- Image pull policies

## Runtime Security

Protecting containers during execution:

- Container runtime security (runsc, kata, etc.)
- Runtime security monitoring
- Threat detection and prevention
- Securing the host OS

## Auditing

Tracking activities in your cluster:

- Kubernetes audit logs
- Audit policy configuration
- Log aggregation and analysis
- Compliance monitoring

## Examples

This directory contains practical examples demonstrating various security features:

1. [Basic RBAC Setup](./examples/01-rbac-basics.yaml) - Creating roles, role bindings, and service accounts
2. [Pod Security Standards](./examples/02-pod-security-standards.yaml) - Implementing different security levels
3. [Security Contexts](./examples/03-security-contexts.yaml) - Configuring security settings for pods
4. [Secure Ingress](./examples/04-secure-ingress.yaml) - Setting up TLS, authentication on Ingress
5. [OPA Gatekeeper Policies](./examples/05-opa-gatekeeper.yaml) - Implementing policy-as-code

## Best Practices

1. **Follow Principle of Least Privilege**
   - Only grant the permissions necessary for the task
   - Regularly audit and prune permissions

2. **Implement Defense in Depth**
   - Use multiple security layers
   - Don't rely on a single security control

3. **Secure Container Images**
   - Use minimal base images
   - Scan for vulnerabilities
   - Sign and verify images

4. **Network Segmentation**
   - Use Network Policies to restrict communication
   - Implement a zero-trust approach

5. **Secure Secrets Management**
   - Use external secrets management when possible
   - Rotate secrets regularly
   - Enable encryption at rest

6. **Regular Security Audits**
   - Perform regular security assessments
   - Use tools like Kubescape, Falco, or Trivy

7. **Keep Kubernetes Updated**
   - Apply security patches promptly
   - Follow security announcements

## Further Reading

- [Kubernetes Security Documentation](https://kubernetes.io/docs/concepts/security/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
- [OWASP Kubernetes Top 10](https://owasp.org/www-project-kubernetes-top-ten/)
- [Kubernetes Security Best Practices](https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/)
- [Cloud Native Security](https://kubernetes.io/docs/concepts/security/overview/) 