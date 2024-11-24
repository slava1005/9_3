## Домашнее задание к занятию «Безопасность в облачных провайдерах»
Используя конфигурации, выполненные в рамках предыдущих домашних заданий, нужно добавить возможность шифрования бакета.

### Задание 1. Yandex Cloud
С помощью ключа в KMS необходимо зашифровать содержимое бакета:
- создать ключ в KMS;
- с помощью ключа зашифровать содержимое бакета, созданного ранее.

### Выполнение задания 1. Yandex Cloud
Для выполнения задания буду использовать код Terraform выполненный в прошлом задании, т.к. в нем уже есть бакет и загрузка изображения в него.
Создаю роль для службы KMS, которая даст возможность зашифровывать и расшифровывать данные:
```
resource "yandex_resourcemanager_folder_iam_member" "sa-editor-encrypter-decrypter" {
  folder_id = var.folder_id
  role      = "kms.keys.encrypterDecrypter"
  member    = "serviceAccount:${yandex_iam_service_account.service.id}"
}
```
Создаю симметричный ключ шифрования с алгоритмом шифрования AES_128 и временем жизни 24 часа:
```
resource "yandex_kms_symmetric_key" "secret-key" {
  name              = "key-1"
  description       = "ключ для шифрования бакета"
  default_algorithm = "AES_128"
  rotation_period   = "24h"
}
```
Применяю ключ шифрования к созданному ранее бакету:
```
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        kms_master_key_id = yandex_kms_symmetric_key.secret-key.id
        sse_algorithm     = "aws:kms"
      }
    }
  }
```
После применения кода Terraform проверю результат:

![img1_1](https://github.com/user-attachments/assets/40580e8c-a1ec-404e-a276-210715589c12)

Ключ шифрования создан.

Открою зашифрованный файл в браузере:

![img1_222](https://github.com/user-attachments/assets/ec97f5d5-6c35-4b4b-b7ce-6ea43b1d4861)

Доступа к файлу в бакете нет, т.к. он зашифрован.
