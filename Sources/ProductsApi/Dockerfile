FROM jcorioland/dotnetcore:1.0.0-preview2-003131

MAINTAINER Julien Corioland, Microsoft (@jcorioland)

WORKDIR /app
ENTRYPOINT ["dotnet", "run"]

ENV ASPNETCORE_URLS http://+:5000
EXPOSE 5000

COPY project.json ./
RUN ["dotnet", "restore"]

COPY . /app
RUN ["dotnet", "build"]