# [BLOCK-1] Стадия загрузки и подготовки
FROM alpine AS preparer
RUN apk add --no-cache wget unzip
WORKDIR /download
RUN wget -q https://storage.yandexcloud.net/intraservice-distr/5.4.7-linux-agent.zip -O site-agent.zip && \
    unzip -q site-agent.zip && \
    rm site-agent.zip

# [BLOCK-2] Финальная стадия
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# [BLOCK-3] Устанавливаем gettext для envsubst
RUN apt-get update && \
    apt-get install -y gettext && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/intraservice.agent

# [BLOCK-4] Копируем содержимое
COPY --from=preparer /download/agent/ .

# [BLOCK-5] ПРОСТАЯ УСТАНОВКА ПРАВ
# Важно: сначала меняем владельца, потом устанавливаем права
RUN chown -R www-data:www-data /var/www/intraservice.agent && \
    chmod -R 755 /var/www/intraservice.agent && \
    chmod 644 /var/www/intraservice.agent/Certificate/*.p8 && \
    chmod 644 /var/www/intraservice.agent/Certificate/*.p12

# [BLOCK-6] Копируем шаблон appsettings.json
COPY appsettings-agent.template.json /appsettings.template.json

EXPOSE 5002

# [BLOCK-7] Запускаем от www-data
USER www-data

# [BLOCK-8] Команда запуска
CMD ["/bin/bash", "-c", "envsubst < /appsettings.template.json > /var/www/intraservice.agent/appsettings.json && \
  echo 'appsettings.json generated successfully' && \
  exec dotnet /var/www/intraservice.agent/IntraService.Agent.Service.dll"]
