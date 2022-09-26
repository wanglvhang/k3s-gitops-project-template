#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 5000
EXPOSE 5001

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["./", "."]
RUN dotnet restore "DemoProject.csproj"
RUN dotnet build "DemoProject.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "DemoProject.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
ENV ASPNETCORE_URLS=http://+:5000
WORKDIR /app
COPY --from=publish /app/publish .

ENTRYPOINT ["dotnet", "DemoProject.dll"]