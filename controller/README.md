# PREEMPT-FaaS Controller

This directory contains the source code and build configuration for the **PREEMPT-FaaS** controller. Follow this documentation to build your own container image of the controller.

## Prerequisites (recommended)

To build the Docker image, you need:

- **Docker** v29.1.3+
- **Docker Buildx** v.0.30.1+
- A **Registry** to store the built image (we report the procedure employing **Docker Hub**)

No local Rust installation is required as the build happens inside a Docker container.

Please refer to the main project docs for prerequisites installation docs links: [README.md](../README.md).

## Project Structure

```
controller/
├── Cargo.toml          # Rust project package and dependencies
├── Cargo.lock          # Locked dependency versions
├── Dockerfile          # Multi-stage Docker build configuration
├── src/                # Controller source code
│   ├── main.rs         # Application entry point
│   ├── components/     # Core controller components
│   └── utils/          # Utility modules
└── target/             # Build artifacts (local builds only)
```

The **target** folder is not present in this repository. It might be automatically created by the **Cargo Rust** tool, if installed. The build process happens in a container, thus, it is not required.

## Building the Docker Image

The controller uses a **multi-stage Docker build** to create a minimal production image:

1. **Build stage**: Compiles the Rust application using `rust:1.91.1-bookworm`
2. **Runtime stage**: Creates a minimal image based on `debian:bookworm-slim` with only the compiled binary

```bash
# Enter the controller folder

# Login to Docker Hub
docker login

# Build and Push the controller image
docker build -t <your-registry-username>/<your-image-name>:<your-image-tag> .
docker push <your-registry-username>/<your-image-name>:<your-image-tag>
```

### Complete Build and Push Workflow Example

Here's a complete example workflow for building and pushing a new version:

```bash
# Set variables
VERSION="1.0.0"
REGISTRY_USER="yourusername"
IMAGE_NAME="preempt-faas"

# Build the image
docker build -t ${REGISTRY_USER}/${IMAGE_NAME}:${VERSION} .

# Tag as latest
docker tag ${REGISTRY_USER}/${IMAGE_NAME}:${VERSION} ${REGISTRY_USER}/${IMAGE_NAME}:latest

# Verify the build
docker images | grep ${REGISTRY_USER}/${IMAGE_NAME}:${VERSION}
docker images | grep ${REGISTRY_USER}/${IMAGE_NAME}:latest

# Login to registry
docker login

# Push both tags
docker push ${REGISTRY_USER}/${IMAGE_NAME}:${VERSION}
docker push ${REGISTRY_USER}/${IMAGE_NAME}:latest

# Verify the push
docker pull ${REGISTRY_USER}/${IMAGE_NAME}:${VERSION}
```

### Optimized Build Time

To speed up builds during development, Docker will cache the dependency fetch step. The Dockerfile is optimized to:

1. Copy `Cargo.toml` and `Cargo.lock` first
2. Fetch dependencies (`cargo fetch`)
3. Copy source code and build

This means dependency fetching is only re-run when `Cargo.toml` changes.

### Image Details

The resulting Docker image:

- **Base image**: `debian:bookworm-slim`
- **Binary location**: `/usr/local/bin/Preempt-K8s`
- **Event queue directory**: `/eventqueue` (created automatically)
- **User**: Runs as `root` (required for real-time scheduling capabilities)
- **Default command**: `Preempt-K8s`

## Updating the Helm Chart

After pushing a new image version, update the Helm chart or the manual installation manifest to use it:

1. Edit [`values.yaml`](../helm/values.yaml):
   ```yaml
   preempt_k8s:
     pod:
       container:
         image:
           repository: <your-registry-username>/<your-image-name>
           tag: <your-image-tag>
   ```

1. Edit [`statefulset.yaml`](../resources/controller-deploy/statefulset.yaml):
   ```yaml
   spec:
     template:
       spec:
         containers:
           - name: <your-container-name>
             image: <your-registry-username>/<your-image-name>:<your-image-tag>
   ```

## Run PREEMPT-FaaS in your K8s Cluster

- Follow the docs for the [Helm installation](../helm/README.md) to automatically deploy the controller in yout production cluster.
- Follow the docs for the [manual installation](../resources/README.md) to manually deploy all the controller resources in yout production cluster.
