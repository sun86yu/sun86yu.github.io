---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: doctrine/migrations
tags:
- doctrine/migrations
- PHP
---

doctrine/migrations
===
thinkphp, laravel 等框架都有自己的 migration 命令行工具，方便项目开发的数据库管理工作。

但如果我们不用框架，想在自己的项目里管理，则可以用 ```doctrine/migrations```

安装
---

***方法一:***

```bash
composer require doctrine/migrations
```

直接:

```bash
vendor/doctrine/migrations/bin/doctrine-migrations list 可看到命令
```

***方法二:***

下载 doctrine-migrations.phar

```bash
https://github.com/doctrine/migrations/releases/download/v1.7.2/doctrine-migrations.phar

php doctrine-migrations.phar list
```

要设置数据库的配置文件或者在命令中传入数据库配置.配置文件要更合适

配置文件 ```migrations.xml``` :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-migrations
        xmlns="http://doctrine-project.org/schemas/migrations/configuration"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://doctrine-project.org/schemas/migrations/configuration http://doctrine-project.org/schemas/migrations/configuration.xsd"
>
    <name>Maintainable Doctrine apps tutorial</name>

    <migrations-namespace>DoctrineMigrations</migrations-namespace>
    <migrations-directory>./migrations</migrations-directory>
</doctrine-migrations>
```

同时在项目根目录下创建新目录 migrations 用来存放 migration 声明文件

根目录下创建 ```migrations-db.php``` 内容是:

```php
return array(
    'driver' => 'pdo_mysql',
    'host' => '127.0.0.1',
    'user' => 'root',
    'password' => 'root',
    'dbname' => 'test'
);
```

测试:

```
php doctrine-migrations.phar migration:status
```

执行成功后会在我们定义的库里创建一个表,名为: ```doctrine_migration_versions```, 里面会记录已经执行了哪些脚本

```bash
php doctrine-migrations.phar migration:generate
```

它会在上面创建的 migrations 目录下新建一个文件,如: ```Version20180529075502.php```

内容如:

```php
<?php declare(strict_types=1);

namespace DoctrineMigrations;

use Doctrine\DBAL\Migrations\AbstractMigration;
use Doctrine\DBAL\Schema\Schema;

/**
 * Auto-generated Migration: Please modify to your needs!
 */
final class Version20180529075502 extends AbstractMigration
{
    public function up(Schema $schema) : void
    {
        // this up() migration is auto-generated, please modify it to your needs

    }

    public function down(Schema $schema) : void
    {
        // this down() migration is auto-generated, please modify it to your needs

    }
}
```

我们要在 up 和 down 里执行相应的代码,如:

```php
<?php declare(strict_types=1);

namespace DoctrineMigrations;

use Doctrine\DBAL\Migrations\AbstractMigration;
use Doctrine\DBAL\Schema\Schema;

/**
 * Auto-generated Migration: Please modify to your needs!
 */
final class Version20180529075502 extends AbstractMigration
{
    public function up(Schema $schema): void
    {
        // this up() migration is auto-generated, please modify it to your needs

        $this->addSql('CREATE TABLE t_user (id INT NOT NULL, name VARCHAR(20) NOT NULL, PRIMARY KEY(id)) ENGINE = InnoDB');

        $users = array(
            array('name' => 'mike', 'id' => 1),
            array('name' => 'jwage', 'id' => 2),
            array('name' => 'ocramius', 'id' => 3),
        );
        foreach ($users as $user) {
            $this->addSql('insert into t_user (id, name) VALUES (:id, :name)', $user);
        }
        $this->addSql('CREATE TABLE addresses (id INT NOT NULL, street VARCHAR(255) NOT NULL, PRIMARY KEY(id)) ENGINE = InnoDB');
    }

    public function down(Schema $schema): void
    {
        // this down() migration is auto-generated, please modify it to your needs

    }
}
```

执行:

```bash
php doctrine-migrations.phar migration:execute 20180529075502
```

参数就是上面文件名后面的版本号.执行成功后数据库里可以看到新建了表，表里插入了数据。
