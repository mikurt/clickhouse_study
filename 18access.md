# Контроль доступа

- Создайте пользователя jhon с паролем «qwerty»
- Создайте роль devs
- Выдайте роли devs права на SELECT на любую таблицу
- Выдайте роль devs пользователю jhon

```
create user john identified by 'qwerty'

create role devs

grant select on *.* to devs

grant devs to john

show grants for john
```

- Предоставьте результаты SELECT из system-таблиц, соответствующих созданным сущностям

<img width="682" height="492" alt="image" src="https://github.com/user-attachments/assets/4e065dc3-d77c-418d-9ae6-8ccf8864f7d4" />

<img width="578" height="172" alt="image" src="https://github.com/user-attachments/assets/57cdd6ac-efec-4055-8af0-5e20b6caebae" />

<img width="1140" height="191" alt="image" src="https://github.com/user-attachments/assets/49563ecd-08ae-44d4-9d0b-5bc3996d44be" />

<img width="1213" height="509" alt="image" src="https://github.com/user-attachments/assets/9ccd8976-0103-4634-b282-74b73119c548" />

