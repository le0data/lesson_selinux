# Работа с SELinux

Для выполнения работы используется Hashicorp Vagrant и Oracle VirtualBox 

Образ используется Almalinux/9 (версия 9.5.20241203) из Vagrant clou

# Запуск Nginx на нестандартном порту (4881) 3-мя разными способами:
- переключатели setsebool;
- добавление нестандартного порта в имеющийся тип;
- формирование и установка модуля SELinux.


Проверяем, что в ОС отключен файервол, конфигурацию установленного NGINX, и режим работы SELinux

![proverka_step1](https://github.com/user-attachments/assets/a0fad548-2ce3-4f2a-9d3c-ba828f63e4a1)

Отображается режим **Enforcing**, который означает, что SELinux будет блокировать запрещенную активность.

## Разрешаем в SELinux работу nginx на порту TCP 4881 с помощью переключателей setsebool

1) Находим в логах (/var/log/audit/audit.log) информацию о блокировании порта

Копируем время, в которое был записан этот лог, и, с помощью утилиты `audit2why` смотрим результат
   
![grep_audit2why](https://github.com/user-attachments/assets/28111ff9-336c-4bea-8180-c0234202ea6c)

Утилита `audit2why` покажет почему трафик блокируется. Исходя из вывода утилиты, мы видим, что нам нужно поменять параметр nis_enabled. 

2) Включим параметр nis_enabled и перезапустим nginx

![setsebool_test](https://github.com/user-attachments/assets/77c137ae-5f76-4760-a4ac-e4be0d32c82d)

Так же поверим работоспособность с помощью утилиты *curl*

![result_sebool](https://github.com/user-attachments/assets/5582d76c-7dc0-4a54-a45e-cfd645c4e1e4)

И так же проверим статус параметра *nis_enabled*

![result_sebool_2](https://github.com/user-attachments/assets/9de23e03-89ce-4b24-9672-a20940d9da7f)

3) Возвращаем запрет работы nginx на порту 4881 обратно.

![result_sebool_finish](https://github.com/user-attachments/assets/67373f3d-55ec-41b5-90e6-af529689bada)

## Разрешаем в SELinux работу nginx на порту TCP 4881 с помощью добавления нестандартного порта в имеющийся тип

1) Ищем имеющийся тип, для http трафика

![2_semanage_1](https://github.com/user-attachments/assets/ce9c4c82-faa2-484f-ab3f-cc3b6c70e2ef)

2) Добавим порт 4881 в тип http_port_t

![2_semanage_2](https://github.com/user-attachments/assets/7d933ded-28e2-479d-a6ce-e9e7194600db)

3) Перезапускаем службу nginx 

![2_semanage_restart_nginx_3](https://github.com/user-attachments/assets/cfc71a6f-7621-4b20-81d3-307c4af5b930)


4) Проверяем работоспособность

![2_semanage_curl_4](https://github.com/user-attachments/assets/f72e7155-5be4-4e69-8ee4-41e855c697d0)


5) Возвращаем запрет работы nginx на порту 4881 обратно, удаляя нестандартный порт 4881 с помощью команды `semanage port -d -t http_port_t -p tcp 4881`  и получаем результат

![2_semanage_finish_5](https://github.com/user-attachments/assets/0b3a1956-b87c-4b49-9a04-828564e8257c)


## Разрешаем в SELinux работу nginx на порту TCP 4881 с помощью формирования и установки модуля SELinux

1) Пробуем снова зупустить nginx

![3_modul_se_1](https://github.com/user-attachments/assets/75e4dbc3-e79c-4eba-80cc-4688ba247c45)

2) Nginx не запустится, так как SELinux продолжает его блокировать. Посмотрим логи SELinux, которые относятся к nginx

![3_module_grep_nginx_2](https://github.com/user-attachments/assets/c098296d-e963-4177-9464-87dde9bf4953)

3) Воспользуемся утилитой `audit2allow` для того, чтобы на основе логов SELinux сделать модуль, разрешающий работу nginx на нестандартном порту

![3_module_se_3](https://github.com/user-attachments/assets/4126875c-c258-4da2-b588-db226b2f5b9d)

4) По результату `audit2allow` сформировал модуль, и сообщил нам команду, с помощью которой можно применить данный модуль и запускаем nginx

![3_module_se_4](https://github.com/user-attachments/assets/03daa5f3-a988-44d3-a391-313ba58731b0)

5) После добавления модуля nginx запустился без ошибок. При использовании модуля изменения сохранятся после перезагрузки. 
Просмотр всех установленных модулей: `semodule -l`
И для  удаления модуля воспользуемсякомандой `semodule -r nginx`

![3_module_se_5_finish](https://github.com/user-attachments/assets/f9c6451e-8992-4ee2-bde0-798ce0325930)








