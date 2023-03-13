# Домашнее задание к занятию 13.3. «Защита сети» - Паромов Роман


### Подготовка к выполнению заданий

1. Подготовка защищаемой системы:

- установите **Suricata**,
- установите **Fail2Ban**.

2. Подготовка системы злоумышленника: установите **nmap** и **thc-hydra** либо скачайте и установите **Kali linux**.

Обе системы должны находится в одной подсети.

------

### Задание 1

Проведите разведку системы и определите, какие сетевые службы запущены на защищаемой системе:

**sudo nmap -sA < ip-адрес >**

**sudo nmap -sT < ip-адрес >**

**sudo nmap -sS < ip-адрес >**

**sudo nmap -sV < ip-адрес >**

По желанию можете поэкспериментировать с опциями: https://nmap.org/man/ru/man-briefoptions.html.


## Пришлось изменить путь в suricata.yaml для suricata.rules, потому что он оказался в /var/lib/suricata/rules. Теперь /var/log/suricata/fast.log показывает что машину скинуруют nmap с разными ключами.

/var/log/suricata/fast.log

```
03/13/2023-20:12:39.845651  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.0.135:55138 -> 192.168.0.133:80
03/13/2023-20:12:39.845651  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.0.135:55138 -> 192.168.0.133:80
03/13/2023-20:14:35.966916  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.0.135:50064 -> 192.168.0.133:5432
03/13/2023-20:14:35.966914  [**] [1:2010939:3] ET SCAN Suspicious inbound to PostgreSQL port 5432 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.0.135:50064 -> 192.168.0.133:5432
03/13/2023-20:14:34.450671  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.0.135:50063 -> 192.168.0.133:1433
03/13/2023-20:14:34.450672  [**] [1:2010935:3] ET SCAN Suspicious inbound to MSSQL port 1433 [**] [Classification: Potentially Bad Traffic] [Priority: 2] {TCP} 192.168.0.135:50063 -> 192.168.0.133:1433
03/13/2023-20:14:14.686837  [**] [1:2024364:4] ET SCAN Possible Nmap User-Agent Observed [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.0.135:35284 -> 192.168.0.133:80
03/13/2023-20:14:14.685872  [**] [1:2009358:6] ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine) [**] [Classification: Web Application Attack] [Priority: 1] {TCP} 192.168.0.135:35268 -> 192.168.0.133:80
```

------

### Задание 2

Проведите атаку на подбор пароля для службы SSH:

**hydra -L users.txt -P pass.txt < ip-адрес > ssh**

1. Настройка **hydra**: 
 
 - создайте два файла: **users.txt** и **pass.txt**;
 - в каждой строчке первого файла должны быть имена пользователей, второго — пароли. В нашем случае это могут быть случайные строки, но ради эксперимента можете добавить имя и пароль существующего пользователя.

Дополнительная информация по **hydra**: https://kali.tools/?p=1847.

2. Включение защиты SSH для Fail2Ban:

-  открыть файл /etc/fail2ban/jail.conf,
-  найти секцию **ssh**,
-  установить **enabled**  в **true**.

## В логах suricata опять же без изменений, а в /var/log/fail2ban.log появилась запись, попытках подключения по ssh и последующим за ним баном

### /var/log/auth.log
```
Mar 12 22:13:12 debian1 sshd[8155]: Failed password for invalid user admin1 from 192.168.0.134 port 35556 ssh2
Mar 12 22:13:12 debian1 sshd[8124]: Failed password for test from 192.168.0.134 port 35402 ssh2
Mar 12 22:13:12 debian1 sshd[8128]: Failed password for test from 192.168.0.134 port 35440 ssh2
Mar 12 22:13:12 debian1 sshd[8127]: Failed password for test from 192.168.0.134 port 35428 ssh2
Mar 12 22:13:12 debian1 sshd[8130]: Failed password for test from 192.168.0.134 port 35462 ssh2
```
/var/log/fail2ban.log
```
2023-03-12 22:13:12,167 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,281 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,282 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,286 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,710 fail2ban.actions        [8053]: NOTICE  [sshd] 192.168.0.134 already banned
2023-03-12 22:13:12,757 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,779 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,860 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,875 fail2ban.filter         [8053]: INFO    [sshd] Found 192.168.0.134 - 2023-03-12 22:13:12
2023-03-12 22:13:12,911 fail2ban.actions        [8053]: NOTICE  [sshd] 192.168.0.134 already banned
```




