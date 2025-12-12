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

# [BLOCK-5] ДЕТАЛЬНАЯ ДИАГНОСТИКА И УСТАНОВКА ПРАВ
RUN echo "=== ДИАГНОСТИКА ===" && \
    # 1. Проверяем текущего пользователя сборки
    echo "1. Пользователь сборки:" && whoami && id && \
    # 2. Проверяем структуру
    echo "2. Структура Certificate:" && \
    ls -la /var/www/intraservice.agent/Certificate/ && \
    stat /var/www/intraservice.agent/Certificate/ && \
    # 3. Проверяем права в цифровом виде
    echo "3. Права в цифровом виде:" && \
    stat -c "%a %n" /var/www/intraservice.agent/Certificate/ && \
    stat -c "%a %n" /var/www/intraservice.agent/Certificate/* && \
    # 4. Меняем владельца на www-data
    echo "4. Меняем владельца на www-data..." && \
    chown -R www-data:www-data /var/www/intraservice.agent && \
    # 5. Устанавливаем ПРАВА НА ДИРЕКТОРИЮ 755 (rwxr-xr-x)
    echo "5. Устанавливаем права на директорию..." && \
    chmod 755 /var/www/intraservice.agent/Certificate/ && \
    # 6. Устанавливаем права на файлы 644 (rw-r--r--)
    chmod 644 /var/www/intraservice.agent/Certificate/*.p8 && \
    chmod 644 /var/www/intraservice.agent/Certificate/*.p12 && \
    # 7. Проверяем результат
    echo "6. Результат:" && \
    ls -la /var/www/intraservice.agent/Certificate/ && \
    stat -c "%a %U:%G %n" /var/www/intraservice.agent/Certificate/ && \
    stat -c "%a %U:%G %n" /var/www/intraservice.agent/Certificate/* && \
    # 8. Проверяем от имени www-data
    echo "7. Проверка от www-data:" && \
    runuser -u www-data -- ls -la /var/www/intraservice.agent/Certificate/ && \
    runuser -u www-data -- test -r /var/www/intraservice.agent/Certificate/AuthKey_AW34Z3FWSW.p8 && echo "Файл читается OK" || echo "Файл НЕ читается" && \
    runuser -u www-data -- test -x /var/www/intraservice.agent/Certificate/ && echo "Директория доступна OK" || echo "Директория НЕ доступна"

# [BLOCK-6] Копируем шаблон appsettings.json
COPY appsettings-agent.template.json /appsettings.template.json

EXPOSE 5002

# [BLOCK-7] Запускаем от www-data
USER www-data

# [BLOCK-8] Команда запуска с максимальной диагностикой
CMD ["/bin/bash", "-c", "echo '=== СТАРТ ПРИЛОЖЕНИЯ ===' && \
  echo '1. Пользователь процесса:' && whoami && id && \
  echo '2. Проверка директории Certificate:' && \
  ls -ld /var/www/intraservice.agent/Certificate/ && \
  echo '3. Содержимое Certificate:' && \
  ls -la /var/www/intraservice.agent/Certificate/ && \
  echo '4. Проверка доступа к директории:' && \
  if [ -r /var/www/intraservice.agent/Certificate/ ]; then echo 'Директория читается'; else echo 'Директория НЕ читается'; fi && \
  if [ -x /var/www/intraservice.agent/Certificate/ ]; then echo 'Директория доступна'; else echo 'Директория НЕ доступна'; fi && \
  echo '5. Попытка чтения файла:' && \
  if [ -r /var/www/intraservice.agent/Certificate/AuthKey_AW34Z3FWSW.p8 ]; then \
    echo 'Файл читается, первые 50 символов:' && \
    head -c 50 /var/www/intraservice.agent/Certificate/AuthKey_AW34Z3FWSW.p8 && echo '...'; \
  else echo 'Файл НЕ читается!'; fi && \
  echo '6. Генерация appsettings.json...' && \
  envsubst < /appsettings.template.json > /var/www/intraservice.agent/appsettings.json && \
  echo 'appsettings.json generated successfully' && \
  echo '7. Запуск .NET приложения...' && \
  exec dotnet /var/www/intraservice.agent/IntraService.Agent.Service.dll"]
