# Стадия загрузки и подготовки
FROM alpine AS preparer
RUN apk add --no-cache wget unzip
WORKDIR /download
RUN wget -q https://storage.yandexcloud.net/intraservice-distr/5.4.7-linux-agent.zip -O site-agent.zip && \
    unzip -q site-agent.zip && \
    rm site-agent.zip && \
    # Устанавливаем правильные права на распакованные файлы
    chmod -R 755 agent/ && \
    # Особое внимание к директории Certificate
    chmod 775 agent/Certificate/

# Финальная стадия
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# Устанавливаем gettext для envsubst
RUN apt-get update && \
    apt-get install -y gettext && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/intraservice.agent

# Копируем содержимое из подготовленной стадии
COPY --from=preparer /download/agent/ .

# УСТАНАВЛИВАЕМ ПРАВИЛЬНОГО ВЛАДЕЛЬЦА ДЛЯ ВСЕХ ФАЙЛОВ
RUN chown -R 1000:1000 /var/www/intraservice.agent && \
    chmod -R 755 /var/www/intraservice.agent && \
    # Особые права для директории Certificate (чтение/запись)
    chmod 775 /var/www/intraservice.agent/Certificate/

# Проверяем права
RUN echo "Проверка прав доступа:" && \
    ls -la /var/www/intraservice.agent/ && \
    echo "Права на Certificate:" && \
    ls -la /var/www/intraservice.agent/Certificate/

# Копируем шаблон appsettings.json
COPY appsettings-agent.template.json /appsettings.template.json

EXPOSE 5002

# Запускаем от непривилегированного пользователя
USER 1000

# Генерируем appsettings.json и запускаем приложение в одной команде
CMD ["/bin/bash", "-c", "envsubst < /appsettings.template.json > /var/www/intraservice.agent/appsettings.json && \
  echo 'appsettings.json generated successfully' && exec dotnet /var/www/intraservice.agent/IntraService.Agent.Service.dll"]
