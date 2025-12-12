# [BLOCK-1] Стадия загрузки и подготовки
FROM alpine AS preparer
RUN apk add --no-cache wget unzip
WORKDIR /download
RUN wget -q https://storage.yandexcloud.net/intraservice-distr/5.4.7-linux-agent.zip -O site-agent.zip && \
    unzip -q site-agent.zip && \
    rm site-agent.zip && \
    # Устанавливаем правильные права НА СЕРТИФИКАТЫ
    chmod 644 agent/Certificate/*.p8 && \
    chmod 644 agent/Certificate/*.p12 && \
    # Устанавливаем права на директорию для ЧТЕНИЯ+ВЫПОЛНЕНИЯ
    chmod 755 agent/Certificate/

# [BLOCK-2] Финальная стадия
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# [BLOCK-3] Устанавливаем gettext для envsubst
RUN apt-get update && \
    apt-get install -y gettext && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /var/www/intraservice.agent

# [BLOCK-4] Копируем содержимое из подготовленной стадии С ПРАВИЛЬНЫМ ВЛАДЕЛЬЦЕМ
COPY --from=preparer /download/agent/ .

# [BLOCK-5] ИСПРАВЛЕНИЕ ПРАВ - КЛЮЧЕВОЙ БЛОК
RUN echo "=== Установка правильных прав ===" && \
    # Меняем владельца на root:root сначала
    chown -R root:root /var/www/intraservice.agent && \
    # Устанавливаем права 755 на все директории и файлы
    chmod -R 755 /var/www/intraservice.agent && \
    # Особые права для файлов сертификатов - 644 (только чтение)
    chmod 644 /var/www/intraservice.agent/Certificate/*.p8 2>/dev/null || echo "Файлы .p8 не найдены" && \
    chmod 644 /var/www/intraservice.agent/Certificate/*.p12 2>/dev/null || echo "Файлы .p12 не найдены" && \
    # Проверяем результат
    echo "=== Проверка прав ===" && \
    ls -la /var/www/intraservice.agent/ && \
    echo "=== Права на Certificate ===" && \
    ls -la /var/www/intraservice.agent/Certificate/

# [BLOCK-6] Копируем шаблон appsettings.json
COPY appsettings-agent.template.json /appsettings.template.json

EXPOSE 5002

# [BLOCK-7] ВАЖНО: НЕ МЕНЯЕМ ПОЛЬЗОВАТЕЛЯ - оставляем root
# USER 1000  # ЗАКОММЕНТИРУЙТЕ ЭТУ СТРОКУ

# [BLOCK-8] Команда запуска
CMD ["/bin/bash", "-c", "envsubst < /appsettings.template.json > /var/www/intraservice.agent/appsettings.json && \
  echo 'appsettings.json generated successfully' && \
  echo '=== Проверка доступа к сертификатам ===' && \
  ls -la /var/www/intraservice.agent/Certificate/ && \
  echo '=== Попытка чтения файла ===' && \
  cat /var/www/intraservice.agent/Certificate/AuthKey_AW34Z3FWSW.p8 | head -c 50 && \
  echo '...' && \
  exec dotnet /var/www/intraservice.agent/IntraService.Agent.Service.dll"]
