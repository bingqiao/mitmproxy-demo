FROM golang:1.22.1-bullseye as build

WORKDIR /app

# Copy the Go module files
COPY go.mod .

# Download the Go module dependencies
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 go build -o /wttr .
 
FROM alpine:latest as run

# Copy the application executable from the build image
COPY --from=build /wttr /wttr

EXPOSE 8080
CMD ["/wttr"]