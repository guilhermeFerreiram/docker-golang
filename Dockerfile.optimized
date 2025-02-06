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
