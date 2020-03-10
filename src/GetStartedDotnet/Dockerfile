FROM mcr.microsoft.com/dotnet/core/aspnet:2.2

WORKDIR /app1

COPY ./bin/Release/netcoreapp2.2/publish/ .

ENTRYPOINT ["dotnet", "GetStartedDotnet.dll"]