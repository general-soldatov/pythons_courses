# создание профиля аккаунта на яндекс

[утилита yc](https://yandex.cloud/ru/docs/cli/quickstart#macos_1)

jq — это популярная консольная утилита, предназначенная для синтаксического анализа, фильтрации и модификации данных в формате JSON. Её часто называют «sed или awk для JSON». Она позволяет легко извлекать нужную информацию из API-ответов прямо в терминале и автоматизировать обработку данных в bash-скриптах.

установим утилиту jq linux
```bash
sudo apt install jq
```
macos
```bash
brew install jq
```
создадим сервисный аккаунт для работы с нашей рабочей станции
```bash
export SERVICE_ACCOUNT=$(yc iam service-account create \
  --name service-account-for-cf \
  --description "service account for cloud functions" \
  --format json | jq -r .)
```
Для проверки текущего списка сервисных аккаунтов можно воспользоваться командой
```bash
yc iam service-account list
```
Информацию по сервисному аккаунту посмотрим с помощью команды:
```bash
echo $SERVICE_ACCOUNT
```
Для хранения идентификатора сервисного аккаунта запишем информацию в переменную окружения
```bash
echo "export SERVICE_ACCOUNT_ID=<идентификатор_сервисного_аккаунта>" >> ~/.bashrc && . ~/.bashrc
echo $SERVICE_ACCOUNT_ID
```
для mac os 
```bash
echo "export SERVICE_ACCOUNT_ID=<идентификатор_сервисного_аккаунта>" >> ~/ ~/.zshrc && . ~/ ~/.zshrc
echo $SERVICE_ACCOUNT_ID
```
назначим роль аккаунту редактора `editor`. Сохраним идентификатор папки, с которой мы работаем в переменные окружения:
```
echo "export FOLDER_ID=$(yc config get folder-id)" >> ~/.bashrc && . ~/.bashrc 
echo $FOLDER_ID
```
А теперь с помощью консольной утилиты назначим роль
```bash
yc resource-manager folder add-access-binding $FOLDER_ID \
  --subject serviceAccount:$SERVICE_ACCOUNT_ID \
  --role editor
```
# создадим функцию
для этого используем команду:
```bash
yc serverless function create --name my-first-function
```
так мы получим url и информацию по нашей функции. По умолчанию она непубличная, т.е. этот url мы не сможем вызвать в браузере.

создадим файл с кодом `index.py`:
```bash
sudo nano index.py
```
И добавим следующий код:
```python
def handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello World!',
    }
```
При обращении к этой функции мы получим небольшую веб-страницу и надписью.

чтобы загрузить этот код на облако, воспользуемся следующей командой (рекомендую сохранить в файл `script.sh`)
```bash
yc serverless function version create \
    --function-name my-first-function \
    --memory 256m \
    --execution-timeout 5s \
    --runtime python312 \
    --entrypoint index.handler \
    --service-account-id $SERVICE_ACCOUNT_ID \
    --source-path index.py
```
Для того, чтобы файл со скриптом запустился, необходимо прописать в терминале разрешение на его запуск:
```bash
sudo chmod a+x script.sh
```
И запустим файл:
```bash
./script.sh
```
Разумеется, мы хотели запустить эту функцию. Для этого получим идентификатор функции. Сделать это можно с помощью команды получения всех функций:
```bash
yc serverless function list
```
Или через обращение к ней через название:
```bash
yc serverless function version list --function-name my-first-function
```
полученный идентификатор функции можем использовать для её вызова:
```bash
yc serverless function invoke <идентификатор_функции>
```
В результате работы этой команды мы можем получить такой ответ:
```
{
  'statusCode': 200,
  'body': 'Hello World!',
}
```
Итак, функция работает! С чем я вас и поздравляю! Но, всё же, хотелось бы по этому url обратиться через браузер. А для этого нам необходимо сделать функцию публичной с помощью команды:
```bash
yc serverless function allow-unauthenticated-invoke my-first-function
```
если url вы забыли, то его можно получить с помощью команды:
```bash
yc serverless function get my-first-function
```
нам интересен параметр `http_invoke_url`, который можно вставить в строку браузера и насладиться созданным первым сайтом))
# набор файлов
чаще программы состоят из нескольких файлов, поэтому для того, чтобы залить код в функцию, можем создать `zip` архив и загрузить весь код в облако:
```bash
yc serverless function version create \
  --function-name my-first-function \
  --memory 256m \
  --execution-timeout 5s \
  --runtime python312 \
  --entrypoint index.handler \
  --service-account-id $SERVICE_ACCOUNT_ID \
  --source-path my-first-function.zip
```
чтобы использовать переменные окружения можем дополнить функцию:
```bash
yc serverless function version create \
  --function-name my-first-function \
  --memory 256m \
  --execution-timeout 5s \
  --runtime python312 \
  --entrypoint index.handler \
  --service-account-id $SERVICE_ACCOUNT_ID \
  --source-version-id <ID> \
  --environment ACCESS_KEY=$ACCESS_KEY \
  --environment SECRET_KEY=$SECRET_KEY \
  --environment BUCKET_NAME=$BUCKET_NAME
```
