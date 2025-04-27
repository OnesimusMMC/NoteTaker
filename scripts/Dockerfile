# Backend build stage
FROM golang:1.24-alpine AS backend
WORKDIR /backend-build

COPY . .
RUN go build -o memos ./bin/memos/main.go

# Frontend build stage
FROM node:18-alpine AS frontend
WORKDIR /frontend-build

COPY ../web ./web
WORKDIR /frontend-build/web

# Install dependencies and build the frontend
RUN npm install -g pnpm && pnpm install && pnpm release

# Final runtime stage
FROM alpine:latest AS monolithic
WORKDIR /usr/local/memos

RUN apk add --no-cache tzdata
ENV TZ="UTC"

# Copy backend binary
COPY --from=backend /backend-build/memos /usr/local/memos/

# Copy frontend static files
COPY --from=frontend /frontend-build/web/dist /usr/local/memos/frontend

# Copy entrypoint script
COPY ./scripts/entrypoint.sh /usr/local/memos/

EXPOSE 5230

# Directory to store the data, which can be referenced as the mounting point.
RUN mkdir -p /var/opt/memos
VOLUME /var/opt/memos

ENV MEMOS_MODE="prod"
ENV MEMOS_PORT="5230"

ENTRYPOINT ["./entrypoint.sh", "./memos"]
