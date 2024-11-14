# Distroless Images & Docker Multi Stage Builds 
### Distroless Images
Distroless images are Docker images built by Google that contain only your application and its runtime dependencies, without a package manager, shell, or other operating system utilities. They are designed to be minimal, secure, and efficient, as they reduce the attack surface and potential vulnerabilities.
### Why do we need Distroless Images?
1. Reduces Attack Surface: Without a shell or other utilities, there are fewer vulnerabilities that an attacker could exploit.
2. Minimizes Image Size: By including only what is necessary for running the application, the image becomes smaller, which can improve load and deployment times.
Distroless images are especially useful in production environments where you want to minimize the size of the image and reduce exposure to security risks. By removing unnecessary components, these images also tend to be smaller and load faster.
