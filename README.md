# PortForwarding

## Общее понятие

PortForwarding — это переброс портов.  

Как оно работает?
Очень просто: в этом методе есть ДВА порта: локальный и публичный.  

Локальный — порт, на котором работает твой сервер, будь то TCP (Minecraft) или HTTP (сайт).  
Публичный — порт на облачном сервере, через который сетевые клиенты будут посещать ресурс.  

При перебросе портов трафик с локального порта напрямую идет на публичный.  
То есть, твой локальный порт «связывается» с публичным портом на удаленном сервере.  

# Конфиг

Запусти программу, она создаст settings.json.  
В конфиге будут дефолтные настройки.  

```JSON
{
  "host": "SERVER_HOST",
  "user": "SERVER_USER",
  "password": "SERVER_PASSWORD",
  "forwarding":
  [
    {
      "RemotePort": 5555,
      "LocalPort": 9999,
      "LocalIp": "127.0.0.1"
    },
    { 
      "RemotePort": 5556,
      "LocalPort": 9998,
      "LocalIp": "127.0.0.1"
    }
  ]
}
```

С host по password относятся к SSH настройкам  

В данном примере существуют уже 2 порта для перенаправления:  

**9999 → 5555**  
**9998 → 5556**

## Добавляем порт

Блок порта выглядит следующим образом  
```JSON
{ 
  "RemotePort": 25565,
  "LocalPort": 25565,
  "LocalIp": "127.0.0.1"
}
```
RemotePort - Публичный порт  
LocalPort - Локальный порт  
LocalIp - Локальный IP у которого будем брать LocalPort  

Упоминаю, 127.0.0.1 - IP компьютера (Не в локальной сети Модема).  

Полный пример  
```JSON
{
  "host": "SERVER_HOST",
  "user": "SERVER_USER",
  "password": "SERVER_PASSWORD",  
  "forwarding":  
  [
    {
      "RemotePort": 5555,  
      "LocalPort": 9999,  
      "LocalIp": "127.0.0.1"  
    },
    { 
      "RemotePort": 5556,
      "LocalPort": 9998,
      "LocalIp": "127.0.0.1"
    },
    { 
      "RemotePort": 25565,
      "LocalPort": 25565,
      "LocalIp": "127.0.0.1"
    }
  ]
}
```
В данном примере, мы добавили переброску локального порта 25565 на тот же порт только у Linux Сервера.  

# Настройка Linux Сервера
В данном примере будем настраивать Ubuntu 22.02 Сервер.  

Создаём пользователя.
Лично, рекомендую создать пользователя без прав.  

```bash
sudo adduser PortForwardingUser # К примеру, такое имя пользователя
# После выше написанной команды попросит пароль для нового аккаунта, при вводе его не будет видно.

sudo usermod -G PortForwardingUser PortForwardingUser # Ограничиваем права 
sudo usermod -s /usr/sbin/nologin PortForwardingUser # Запрещаем пользователю входить в систему по дефолтному SSH 
```
Хорошо, мы создали пользователя без прав. Он не может входить в систему.  

### Теперь нужно настроить sshd сервис.  
```bash
sudo nano /etc/ssh/sshd_config
```
Добавляем строчки:  
```conf
PermitTunnel yes
GatewayPorts yes
```
PermitTunnel - этот параметр отвечает за Layer 3  
GatewayPorts - этот параметр отвечает за то, что ваши перебросанные порты, будут не только в локальной сети сервера, но и доступны из интернета.  

Далее добавляем
```conf
Match User PortForwardingUser
    AllowTcpForwarding yes
```

Готово! Теперь вводим в конфиг 
host - Айпи Ubuntu Сервера  
user - PortForwardingUser  
password - Пароль от ssh аккаунта  

# Заключение
Теперь после этих процедур, ваши локальные порты будут доступны в интернете.  
Главное, чтобы на Ubuntu сервере был публичный IP, обычно, хостинги например, Timeweb Cloud предоставляет публичный IP за 150 р к стоимости сервера.  
  
  
# Ошибки и исключения
`The list of ports is empty.` - Начиная с версии 1.2, пустой список портов не допустим, добавьте хотя-бы одно пребрасывание порта.  
`Error Types In Settings` - Одно или несколько значение(ий) "RemotePort" и(или) не являеться [integer типом](https://ru.wikipedia.org/wiki/%D0%A6%D0%B5%D0%BB%D0%BE%D0%B5_(%D1%82%D0%B8%D0%BF_%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85)).  
`Error While Read settings.json` - Произошла ошибка при чтении `settings.json`, поробуйте удалить, запустить программу, и заполнить заного.  
`Error While Parsing settings.json` - Действия теже что у `Error While Read settings.json`  
`ERROR: Host is null` - Параметр `host` не найден.  
`ERROR: User is null` - Параметр `user` не найден.  
`ERROR: Password is null` - Параметр `password` не найден.  
`ERROR: Forwarding is null` - Список портов не найден.  
