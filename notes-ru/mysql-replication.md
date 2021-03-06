## Особенности репликации и бэкапов в MySQL

Есть такая чудесная и широко известная утилита для снятия бэкапов с MySQL - [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup). Снять бэкап с запущенного сервера с её помощью довольно просто:

```bash
BACKUP_NAME='dump'
innobackupex --rsync --no-timestamp $BACKUP_NAME
innobackupex --rsync --no-timestamp --apply-log $BACKUP_NAME
tar --use-compress-program=pigz -cpf $BACKUP_NAME.tar.gz $BACKUP_NAME
rm -r $BACKUP_NAME
```

Первая команда создает неповрежденную копию файлов, вторая -- записывает нужную позицию в логфайле.

Статьи по развертыванию реплики MySQL-сервера по большей части делятся на две категории:
1. [Остановите сервер и просто скопируйте файлы](https://ruhighload.com/post/Как+настроить+MySQL+Master-Slave+репликацию). Если есть такая возможность -- делайте так. Но обычно её нет.
2. [Останавливать ничего не надо, используем innobackupex](https://ruhighload.com/post/Репликация+без+простоя).

### Одна очень важная деталь

Для того, чтобы ежедневные (а то и ежечасные) бэкапы не блокировали таблицы на мастере, для создания бэкапов лучше всего использовать отдельную реплику. Однако такие бэкапы могут оказаться невалидны для создания другой реплики. И даже если реплику остановить и снять "холодный" дамп. Для создания реплики всегда нужно использовать дамп, созданный с мастера.
