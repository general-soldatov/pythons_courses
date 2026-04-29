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
