# Optimizing Docker Image Size
This repository demonstrates how to reduce the size of a Docker image by optimizing its build process. The base image starts at 1GB, and through a series of steps, we aim to achieve a much smaller, ultra-minimal Docker image that contains only the necessary components for running the application.

## Base Image (1GB)
```
# Base Image
FROM golang:latest

# Set the Current Working Directory inside the container
WORKDIR /app

# Copy everything from the current directory to the PWD(Present Working Directory) inside the container
COPY . .

# Build the Go app
RUN go build -o main .

# Expose port 8080 to the outside world
EXPOSE 8080

# Command to run the executable
CMD ["./main"]

```

## Optimized Image (2MB)
In this optimized version, we follow a two-step approach:

1. Build the Go binary: We build the Go binary using a multi-stage Dockerfile. This stage uses a golang:latest image to compile the application, then it installs UPX (Ultimate Packer for eXecutables) to further reduce the binary size.

2. Ultra-minimal final image: We use the scratch base image, which is essentially an empty image. This ensures that only the compiled binary is included in the final Docker image.

```
# Step 1: Build the binary
FROM golang:latest AS builder

WORKDIR /app
COPY . .

# Install UPX
RUN apt-get update && apt-get install -y upx

# Compile the binary with optimization and make it static
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o main .

# Compress the binary with UPX to reduce size
RUN upx --best --lzma main

# Step 2: Ultra-minimal final image
FROM scratch

WORKDIR /root/
COPY --from=builder /app/main .

# Expose the required port
EXPOSE 8080

# Run the application
CMD ["./main"]
```

## Results
By following these steps, the size of the Docker image is significantly reduced, potentially going from 1GB to just a few megabytes, depending on the size of the Go binary.

## Running the Example
1. Clone this repository:
```
git clone https://github.com/guilhermeFerreiram/docker-golang.git
```

2. Build and check the size of the base image:
```
docker build -t base-image -f Dockerfile.base .
docker images base-image
```

3. Build and check the size of the optimized image:
```
docker build -t optimized-image -f Dockerfile.optimized .
docker images optimized-image
```

4. Run the optimized image:
```
docker run --rm -p 8080:8080 optimized-image
```

5. See the terminal output:
```
Hello, World!
```
