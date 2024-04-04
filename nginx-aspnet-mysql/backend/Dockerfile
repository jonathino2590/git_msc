# syntax=docker/dockerfile:1.4

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:6.0 AS base

WORKDIR /opt

COPY aspnetapp.csproj ./
RUN ["dotnet", "restore"]

FROM base as produccion

COPY . .

COPY package.json ./

CMD ["dotnet", "build", "-c", "-o", "/build"]

FROM produccion as dev-envs

RUN <<EOF
apt-get update
apt-get install -y git vim
EOF

RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /

CMD ["dotnet", "start"]

FROM produccion AS publisher

EXPOSE 8080

RUN ["dotnet", "publish", "-c", "Release", "-o", "/build"]

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/aspnet:6.0

WORKDIR /app
COPY --from=publisher /build .

CMD ["dotnet", "aspnetapp.dll"]
