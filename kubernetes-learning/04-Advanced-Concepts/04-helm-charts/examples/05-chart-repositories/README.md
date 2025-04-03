# Helm Chart Repositories

This example demonstrates how to work with Helm chart repositories, including how to create, manage, and publish your own repositories.

## Chart Repository Overview

A chart repository is a location where packaged charts can be stored and shared. It's essentially an HTTP server that hosts an index file and optionally some packaged charts.

## Repository Types

### Public Repositories

Public repositories are accessible to everyone and include:

- **Artifact Hub**: A central location for discovering and sharing Kubernetes packages (https://artifacthub.io/)
- **Bitnami**: Contains common application charts (https://charts.bitnami.com/bitnami)
- **Helm Stable**: The original community chart repository (deprecated, but relevant historically)

### Private Repositories

Private repositories are restricted to authorized users and include:

- **Harbor**: An open-source registry that can store Helm charts
- **ChartMuseum**: A Helm chart repository server
- **Nexus Repository Manager**: A repository manager that supports Helm charts
- **JFrog Artifactory**: A universal artifact repository that supports Helm charts
- **GitHub/GitLab Pages**: Static website hosting that can serve Helm charts
- **Cloud Provider Solutions**: AWS ECR, Azure Container Registry, Google Artifact Registry

## Repository Structure

A basic chart repository consists of:

- **Packaged Charts**: Compressed `.tgz` files (e.g., `mychart-0.1.0.tgz`)
- **index.yaml**: A file containing metadata about the charts in the repository

## Creating a Chart Repository

### Basic Local Repository

1. **Create a directory for your repository**:
   ```bash
   mkdir my-charts
   cd my-charts
   ```

2. **Package a chart**:
   ```bash
   helm package ../path/to/your/chart
   ```

3. **Create or update the index file**:
   ```bash
   helm repo index --url https://example.com/charts .
   ```

4. **Serve the repository**:
   ```bash
   # Using a simple HTTP server
   python -m http.server 8080
   ```

### Setting Up ChartMuseum

ChartMuseum is a dedicated Helm chart repository server:

1. **Install ChartMuseum**:
   ```bash
   # Using Helm
   helm repo add chartmuseum https://chartmuseum.github.io/charts
   helm install my-chartmuseum chartmuseum/chartmuseum --set env.open.DISABLE_API=false
   ```

2. **Configure storage**:
   
   ChartMuseum supports various backend storage providers:
   
   ```yaml
   # Example values for AWS S3 storage
   env:
     open:
       STORAGE: amazon
       STORAGE_AMAZON_BUCKET: my-helm-charts
       STORAGE_AMAZON_REGION: us-east-1
       DISABLE_API: false
   ```

3. **Push charts to ChartMuseum**:
   ```bash
   # Using Helm Push plugin
   helm plugin install https://github.com/chartmuseum/helm-push.git
   helm push mychart-0.1.0.tgz my-repo
   ```

### Using GitHub Pages

GitHub Pages provides a free way to host static content, making it suitable for Helm charts:

1. **Create a GitHub repository**

2. **Set up a `gh-pages` branch**:
   ```bash
   git checkout --orphan gh-pages
   git rm -rf .
   touch index.html
   git add index.html
   git commit -m "Initial commit for GitHub Pages"
   git push origin gh-pages
   ```

3. **Enable GitHub Pages** in repository settings

4. **Create a chart repository structure**:
   ```bash
   mkdir -p charts
   helm package ../path/to/your/chart -d charts/
   helm repo index --url https://username.github.io/repo-name/charts charts/
   git add .
   git commit -m "Add Helm chart repository"
   git push origin gh-pages
   ```

### Using Harbor

Harbor is an open-source container registry that also supports Helm charts:

1. **Install Harbor** (using Helm):
   ```bash
   helm repo add harbor https://helm.goharbor.io
   helm install my-harbor harbor/harbor --set expose.type=nodePort,externalURL=https://harbor.example.com
   ```

2. **Create a project** in the Harbor UI

3. **Configure Helm CLI**:
   ```bash
   helm registry login -u username harbor.example.com
   ```

4. **Push charts to Harbor**:
   ```bash
   helm push mychart-0.1.0.tgz oci://harbor.example.com/project-name
   ```

## Working with Chart Repositories

### Adding a Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### Listing Repositories

```bash
helm repo list
```

### Updating Repositories

```bash
helm repo update
```

### Removing a Repository

```bash
helm repo remove bitnami
```

### Searching for Charts

```bash
helm search repo nginx
```

### Installing Charts from a Repository

```bash
helm install my-release bitnami/nginx
```

## OCI-Based Repositories

Helm supports OCI (Open Container Initiative) registries for storing and distributing charts:

1. **Logging in to an OCI registry**:
   ```bash
   helm registry login registry.example.com
   ```

2. **Pushing a chart to an OCI registry**:
   ```bash
   helm push mychart-0.1.0.tgz oci://registry.example.com/charts
   ```

3. **Pulling a chart from an OCI registry**:
   ```bash
   helm pull oci://registry.example.com/charts/mychart --version 0.1.0
   ```

4. **Installing from an OCI registry**:
   ```bash
   helm install my-release oci://registry.example.com/charts/mychart --version 0.1.0
   ```

## Chart Repository Management

### Versioning

Follow semantic versioning (SemVer) for your charts:

```yaml
# Chart.yaml
apiVersion: v2
name: mychart
version: 1.2.3  # Major.Minor.Patch
```

### Signing Charts

Helm supports chart signing for security:

1. **Create a key pair**:
   ```bash
   gpg --full-generate-key
   ```

2. **Export the public key**:
   ```bash
   gpg --export-secret-keys >~/.gnupg/secring.gpg
   ```

3. **Sign a package**:
   ```bash
   helm package --sign --key 'your-key-name' --keyring ~/.gnupg/secring.gpg chart/
   ```

4. **Verify a package**:
   ```bash
   helm verify chartname-version.tgz
   ```

### Repository Maintenance

Regular tasks for maintaining repositories:

1. **Reindexing charts**:
   ```bash
   helm repo index --url https://example.com/charts --merge index.yaml .
   ```

2. **Cleaning old versions**:
   Implement a retention policy to remove outdated chart versions

3. **Backing up repository data**:
   Set up backups for your repository storage

## Authentication and Authorization

### Basic Authentication

For a simple HTTP server:

```nginx
# Nginx configuration example
server {
    listen 80;
    server_name charts.example.com;
    
    auth_basic "Helm Chart Repository";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    location / {
        root /path/to/charts;
        autoindex on;
    }
}
```

### TLS/SSL

Always use HTTPS for production repositories:

```bash
# Create an Nginx config with SSL
server {
    listen 443 ssl;
    server_name charts.example.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        root /path/to/charts;
        autoindex on;
    }
}
```

## CI/CD Pipeline Integration

Example GitHub Actions workflow for publishing charts:

```yaml
name: Release Charts

on:
  push:
    branches:
      - main
    paths:
      - 'charts/**'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Install Helm
        uses: azure/setup-helm@v1
        with:
          version: v3.8.1

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.4.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

## Advanced Repository Configuration

### Mirroring Repositories

Create a mirror of an existing repository:

```bash
# Download all charts from a repository
mkdir mirror
for chart in $(helm search repo bitnami -o json | jq -r '.[].name'); do
  helm pull $chart --destination mirror
done

# Create index for the mirror
helm repo index --url https://mirror.example.com mirror
```

### High Availability

For production environments, ensure high availability:

1. **Load balancing** across multiple instances
2. **Content delivery networks** (CDNs) for global distribution
3. **Redundant storage** for chart packages
4. **Database replication** for repository metadata (if using a database-backed solution like ChartMuseum)

## Best Practices

1. **Use a consistent naming convention** for charts and versions
2. **Document dependencies** and requirements clearly
3. **Implement CI/CD pipelines** for testing and publishing charts
4. **Version control your charts** in a Git repository
5. **Set up automated testing** for chart validation
6. **Use HTTPS** for all repositories
7. **Implement access controls** for private repositories
8. **Regularly update and maintain** your repository
9. **Monitor repository usage and performance**
10. **Create a clear deprecation policy** for outdated charts

## Troubleshooting

### Common Issues

1. **Repository not accessible**:
   ```bash
   # Check connectivity
   curl -v https://charts.example.com/index.yaml
   ```

2. **Authentication failures**:
   ```bash
   # Update credentials
   helm repo add myrepo https://charts.example.com --username user --password pass
   ```

3. **Chart not found after update**:
   ```bash
   # Force repository update
   helm repo update --force-update
   ```

4. **Index not updating**:
   ```bash
   # Regenerate the index from scratch
   helm repo index --url https://charts.example.com .
   ```

5. **Push failures to ChartMuseum**:
   ```bash
   # Check API status
   curl -v https://chartmuseum.example.com/api/charts
   ```

## Examples

### Deploying a Complete Chart Repository with ChartMuseum

```yaml
# values.yaml for ChartMuseum Helm chart
replicaCount: 2

service:
  type: ClusterIP

ingress:
  enabled: true
  hosts:
    - host: charts.example.com
      paths: ["/"]
  tls:
    - secretName: charts-tls
      hosts:
        - charts.example.com

persistence:
  enabled: true
  size: 10Gi

env:
  open:
    STORAGE: amazon
    STORAGE_AMAZON_BUCKET: helm-charts-bucket
    STORAGE_AMAZON_PREFIX: charts
    STORAGE_AMAZON_REGION: us-east-1
    BASIC_AUTH_USER: admin
    DISABLE_API: false
  secret:
    BASIC_AUTH_PASS: password
    AWS_ACCESS_KEY_ID: your-access-key
    AWS_SECRET_ACCESS_KEY: your-secret-key
```

### Complete GitHub Pages Repository Setup Script

```bash
#!/bin/bash
# Script to set up a Helm chart repository on GitHub Pages

# Configuration
GITHUB_USER="your-username"
REPO_NAME="helm-charts"
CHARTS_DIR="charts"

# Create a new repository on GitHub first, then clone it
git clone https://github.com/$GITHUB_USER/$REPO_NAME.git
cd $REPO_NAME

# Create and switch to gh-pages branch
git checkout --orphan gh-pages
git rm -rf .
touch index.html
echo "<html><body><h1>Helm Charts Repository</h1></body></html>" > index.html
git add index.html
git commit -m "Initial commit for GitHub Pages"
git push origin gh-pages

# Create charts directory
mkdir -p $CHARTS_DIR

# If you have existing charts, package them
# helm package ../path/to/your/chart -d $CHARTS_DIR/

# Create or update the index file
helm repo index --url https://$GITHUB_USER.github.io/$REPO_NAME/$CHARTS_DIR $CHARTS_DIR/

# Commit and push
git add .
git commit -m "Add Helm chart repository"
git push origin gh-pages

# Add the repository to your local Helm
helm repo add my-github-charts https://$GITHUB_USER.github.io/$REPO_NAME/$CHARTS_DIR
```

## Conclusion

Setting up a Helm chart repository is an essential step for organizations developing and maintaining multiple Kubernetes applications. By following the practices outlined in this guide, you can create a reliable, secure, and maintainable repository for your Helm charts, enabling efficient distribution and versioning of your Kubernetes applications. 