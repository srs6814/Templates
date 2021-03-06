# To build this Dockerfile, run the following command from the solution directory:
# docker build --file "Source\ApiTemplate\Dockerfile" --tag api .
# Or you can use the experimental buildx command for a better experience:
# docker buildx build --progress plain --file "Source\ApiTemplate\Dockerfile" --tag api .

# SDK image used to build and publish the application
FROM mcr.microsoft.com/dotnet/core/sdk:3.1-alpine AS sdk
# To use the debug build configuration pass --build-arg Configuration=Debug
ARG Configuration=Release
ENV DOTNET_CLI_TELEMETRY_OPTOUT=true \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true
WORKDIR /src
COPY "ApiTemplate.sln" "."
COPY "Source/ApiTemplate/*.csproj" "Source/ApiTemplate/"
COPY "Tests/ApiTemplate.IntegrationTest/*.csproj" "Tests/ApiTemplate.IntegrationTest/"
RUN dotnet restore
COPY . .
# To view the files that have been copied into the container file system for debugging purposes uncomment this line
# RUN apk add --no-cache tree && tree
RUN dotnet build --configuration $Configuration --no-restore
RUN dotnet test --configuration $Configuration --no-build
RUN dotnet publish "Source/ApiTemplate/ApiTemplate.csproj" --configuration $Configuration --no-build --output /app

# Runtime image used to run the application
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-alpine AS runtime
# Open Container Initiative (OCI) labels (See https://github.com/opencontainers/image-spec/blob/master/annotations.md).
LABEL org.opencontainers.image.title="PROJECT-TITLE" \
      org.opencontainers.image.description="PROJECT-DESCRIPTION" \
      org.opencontainers.image.vendor="PROJECT-AUTHOR"
# Disable the culture invariant mode which defaults to true in the base alpine image
# (See https://github.com/dotnet/corefx/blob/8245ee1e8f6063ccc7a3a60cafe821d29e85b02f/Documentation/architecture/globalization-invariant-mode.md)
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false
RUN apk add --no-cache \
# Install cultures to enable use of System.CultureInfo
    icu-libs \
# Install time zone database to enable use of System.TimeZoneInfo
    tzdata
WORKDIR /app
#if (HttpsEverywhere)
EXPOSE 80 443
##else
#EXPOSE 80
#endif
COPY --from=sdk /app .
ENTRYPOINT ["dotnet", "ApiTemplate.dll"]
