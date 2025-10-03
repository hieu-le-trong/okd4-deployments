# Toge Test Application Helm Chart

This Helm chart deploys the Toge Next.js application to a Kubernetes cluster using the generic-chart as a dependency.

## Application Overview

The Toge application is a Next.js application that:
- Runs on Node.js 18
- Listens on port 3000
- Includes health checks at `/api/health`
- Is built with production optimizations
- Follows OpenShift security best practices

## Prerequisites

- Kubernetes cluster with Helm 3.x installed
- Access to the `hieuletrong368/toge:1.0` Docker image
- DNS configuration for `toge.baloise.dev`

## Installation

1. Add the generic-chart repository:
```bash
helm repo add baloise-incubator https://baloise-incubator.github.io/generic-chart/
helm repo update
```

2. Install the chart:
```bash
helm install toge-test . -n your-namespace
```

3. Upgrade the chart:
```bash
helm upgrade toge-test . -n your-namespace
```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `app.image.repository` | Container image repository | `hieuletrong368/toge` |
| `app.image.tag` | Container image tag | `1.0` |
| `app.network.http.ingress.host` | Ingress hostname | `toge.baloise.dev` |
| `app.deployment.replicas` | Number of replicas | `1` |
| `app.nginx.enabled` | Enable nginx reverse proxy | `false` |
| `app.monitoring.enabled` | Enable ServiceMonitor | `false` |

### Security Features

- Runs as non-root user (UID 1001)
- Drops all Linux capabilities
- Uses read-only root filesystem where possible
- Implements proper OpenShift security contexts

### Health Checks

- **Liveness Probe**: Checks `/api/health` every 10 seconds
- **Readiness Probe**: Checks `/api/health` every 5 seconds
- **Initial Delay**: 30 seconds for liveness, 5 seconds for readiness

## Architecture Options

### Option 1: Direct Next.js Deployment (Current)
- Next.js application runs directly in the container
- Exposed on port 3000
- Ingress routes traffic directly to Next.js

### Option 2: Nginx + Next.js (Optional)
- Enable nginx reverse proxy by setting `app.nginx.enabled: true`
- Nginx handles static content and proxies API requests
- Better for high-traffic scenarios

## Dockerfile Analysis

The provided Dockerfile:
- Uses multi-stage build for optimization
- Creates non-root user following OpenShift best practices
- Implements proper file permissions for group access
- Includes health checks
- Sets appropriate environment variables

## Nginx Configuration

The provided nginx.conf is configured for:
- Static file serving with caching
- Security headers
- Gzip compression
- Health check endpoint
- Can be used as a reverse proxy if needed

## DNS Configuration

The chart includes a DNSEndpoint resource for external-dns integration:
- Automatically manages DNS records for the ingress host
- Points to the ingress controller IP
- TTL set to 180 seconds

## Monitoring

Optional ServiceMonitor can be enabled for Prometheus scraping:
- Scrapes `/api/metrics` endpoint
- 30-second interval
- Requires the application to expose metrics

## Troubleshooting

1. **Health Check Failures**: Ensure your Next.js app has a `/api/health` endpoint
2. **Permission Issues**: Verify the container runs as UID 1001
3. **DNS Issues**: Check that external-dns is configured in your cluster
4. **Image Pull Issues**: Verify access to the container registry

## Development

To test changes locally:
```bash
helm template toge-test . --debug
helm lint .
helm dependency update
```
