FROM microsoft/windowsservercore:latest

MAINTAINER Julien Corioland, Microsoft (@jcorioland)

RUN powershell -Command \
        Invoke-WebRequest 'https://dotnetcli.blob.core.windows.net/dotnet/preview/Binaries/1.0.0-preview2-003131/dotnet-dev-win-x64.1.0.0-preview2-003131.zip' -OutFile dotnet.zip; \
        Expand-Archive dotnet.zip -DestinationPath '%ProgramFiles%\dotnet'; \
        Remove-Item -Force dotnet.zip

RUN setx /M PATH "%PATH%;%ProgramFiles%\dotnet"

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip
RUN mkdir warmup \
    && cd warmup \
    && dotnet new \
    && cd .. \
    && rmdir /q/s warmup