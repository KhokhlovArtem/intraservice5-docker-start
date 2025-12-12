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

# [BLOCK-5] УСТАНАВЛИВАЕМ ПРАВА ДЛЯ ПОЛЬЗОВАТЕЛЯ WWW-DATA (UID 33)
RUN echo "=== Установка прав для www-data (UID 33) ===" && \
    # Проверяем существующих пользователей
    echo "Существующие пользователи:" && \
    grep -E "(root|www-data)" /etc/passwd && \
    # Проверяем текущие права
    echo "Текущие права на Certificate:" && \
    ls -la /var/www/intraservice.agent/Certificate/ && \
    # Меняем владельца на www-data:www-data
    chown -R www-data:www-data /var/www/intraservice.agent && \
    # Устанавливаем правильные права
    find /var/www/intraservice.agent -type d -exec chmod 755 {} \; && \
    find /var/www/intraservice.agent -type f -exec chmod 644 {} \; && \
    # Проверяем результат от имени www-data
    echo "Проверка доступа от www-data:" && \
    sudo -u www-data ls -la /var/www/intraservice.agent/Certificate/ && \
    sudo -u www-data cat /var/www/intraservice.agent/Certificate/AuthKey_AW34Z3FWSW.p8 | head -c 50 && echo "...OK"

# [BLOCK-6] Копируем шаблон appsettings.json
COPY appsettings-agent.template.json /appsettings.template.json

EXPOSE 5002

# [BLOCK-7] Запускаем от пользователя www-data
USER www-data

# [BLOCK-8] Команда запуска
CMD ["/bin/bash", "-c", "echo '=== Запуск от пользователя:' && whoami && echo 'UID:' && id -u && \
  envsubst < /appsettings.template.json > /var/www/intraservice.agent/appsettings.json && \
  echo 'appsettings.json generated successfully' && \
  exec dotnet /var/www/intraservice.agent/IntraService.Agent.Service.dll"]
