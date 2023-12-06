# eShop Reference Application - "Northern Mountains"

A reference .NET application implementing an eCommerce web site using a services-based architecture.

![eShop Reference Application architecture diagram](img/eshop_architecture.png)

![eShop homepage screenshot](img/eshop_homepage.png)

## Getting Started

### Prerequisites

- Clone the eShop repository: https://github.com/dotnet/eshop
- (Windows only) Install Visual Studio. Visual Studio contains tooling support for .NET Aspire that you will want to have. [Visual Studio 2022 version 17.9 Preview](https://visualstudio.microsoft.com/vs/preview/).
  - During installation, ensure that the following are selected:
    - `ASP.NET and web development` workload.
    - `.NET Aspire SDK` component in `Individual components`.
- Install the latest [.NET 8 SDK](https://github.com/dotnet/installer#installers-and-binaries)
- On Mac/Linux (or if not using Visual Studio), install the Aspire workload with the following commands:
```powershell
dotnet workload update
dotnet workload install aspire
dotnet restore eShop.Web.slnf
```
- Install & start Docker Desktop:  https://docs.docker.com/engine/install/

### Running the solution

> [!WARNING]
> Remember to ensure that Docker is started

* (Windows only) Run the application from Visual Studio:
 - Open the `eShop.Web.slnf` file in Visual Studio
 - Ensure that `eShop.AppHost.csproj` is your startup project
 - Hit Ctrl-F5 to launch Aspire

* Or run the application from your terminal:
```powershell
dotnet run --project src/eShop.AppHost/eShop.AppHost.csproj
```
then look for lines like this in the console output in order to find the URL to open the Aspire dashboard:
```sh
Now listening on: http://localhost:18848
```
### Build Images and AKS manifests
> [!WARNING]
> Dependency on [https://github.com/prom3theu5/aspirational-manifests](https://github.com/prom3theu5/aspirational-manifests) to build AKS manifest and docker images for this project
1. Deploy a local registry ([link](https://hub.docker.com/_/registry))  
  ```sh
    docker run -d -p 5001:5000 --restart always --name registry registry:2
  ```
2. Create manifest using aspirate cli using as registry `localhost:5001`
2. Use custom postgreSQL image to use the `pgvector` extension
   1. Follow instructions [here](https://github.com/pgvector/pgvector#docker) with the PostgreSQL version = 12
   2. Build the images
      ```sh
        docker build --build-arg PG_MAJOR=12 -t pgvector .
      ```
   3. Tag the image 
      ```sh
        docker tag pgvector localhost:5001/pgvector
      ```
   4. Push images to the remote repository 
      ```sh
        docker push localhost:5001/pgvector:latest
      ```
3. replace the image use in postgreSQL manifest [postgres-server.yml](.\src\eShop.AppHost\aspirate-output\postgres\postgres-server.yml) by `localhost:5001/pgvector:latest`
   ```yml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: postgres-statefulset
      labels:
        app: postgres
    spec:
      serviceName: "postgres"
      replicas: 1
      selector:
        matchLabels:
          app: postgres
      template:
        metadata:
          labels:
            app: postgres
        spec:
          containers:
            - name: postgres
              image: localhost:5001/pgvector:latest
              envFrom:
                - configMapRef:
                    name: postgres-configuration
              ports:
                - containerPort: 5432
                  name: postgresdb
   ```
4. Go in the [aspirate-outputs](.\src\eShop.AppHost\aspirate-outputs) directory and run 
   ```sh
    kubectl apply -k .\
   ```
### Sample data

The sample catalog data is defined in [catalog.json](https://github.com/dotnet/eShop/blob/main/src/Catalog.API/Setup/catalog.json). Those product names, descriptions, and brand names are fictional and were generated using [GPT-35-Turbo](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/chatgpt), and the corresponding [product images](https://github.com/dotnet/eShop/tree/main/src/Catalog.API/Pics) were generated using [DALL·E 3](https://openai.com/dall-e-3).

### Contributing

For more information on contributing to this repo, please read [the contribution documentation](./CONTRIBUTING.md) and [the Code of Conduct](CODE-OF-CONDUCT.md).
