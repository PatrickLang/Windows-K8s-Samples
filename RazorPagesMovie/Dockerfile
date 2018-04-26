FROM microsoft/dotnet:2.0-sdk AS build
WORKDIR /app/aspnetapp

# copy csproj and restore as distinct layers
# COPY *.sln .
RUN mkdir \db
COPY ./*.csproj .
RUN dotnet restore

# copy everything else and build app
COPY . /app/aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -o out /p:PublishWithAspNetCoreTargetManifest="false"


FROM microsoft/dotnet:2.0-runtime AS runtime
ENV ASPNETCORE_URLS http://+:80
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "RazorPagesMovie.dll"]
