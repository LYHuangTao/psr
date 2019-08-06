# [PSR-4: Autoloader](https://www.php-fig.org/psr/psr-4/)

## 1.概览

此PSR描述了从文件路径自动加载类的规范。
它是完全互操作的，除了PSR-0内的任何其他自动加载规范外还可以使用它。
此PSR也描述了根据规范放置将被自动加载的文件的位置。

## 2.规范

1. 术语"class"指代类（classes）接口（interface），traits和其它相似的结构。
2. 完全限定的类名具有以下形式：

   `\<NamespaceName>(\<SubNamespaceNames>\)*\<ClassName>`

   1. 完全限定的类名MUST（必须）具有一个顶级命名空间名，也被称为“**供应商命名空间（Vender namespace）**”。
   2. 完全限定的类名MAY（可以）具有一个或多个子命名空间。
   3. 完全限定的类名MUST（必须）具有终止类名。
   4. 完全限定的类名中的下划线“_”没有特殊含义。
   5. 完全限定的类名中的英文字母字符可以是小写和大写的任意组合。
   6. MUST（必须）以区分大小写的方式引用所有类名。
3. 加载完全限定的类名对应的文件：

   1. 在完全限定的类名中的一个或多个前导命名空间和子命名空间名称的连续系列（不包括签到命名空间分隔符）即“**命名空间前缀**”对应至少一个“**基目录**”。
   2. “**命名空间前缀**”之后的连续子命名空间名对应“**基目录**”中的子目录，其中命名空间分隔符表示目录分隔符。子目录名称MUST（必须）与子命名空间名称大小写相匹配。
   3. 终止类名对应一个以“**.php**”结尾的文件。文件名MUST（必须）与终止类名大小写相匹配。

4. 自动加载器的实现MUST NOT（禁止）抛出异常，MUST NOT（禁止）触发任何级别的错误并且SHOULD NOT（不应该）返回值。

## 3.示例

下表展示了给定完全限定的类名，命名空间前缀和基础目录对应的文件路径。

| FULLY QUALIFIED CLASS NAME（完全限定的类名） | NAMESPACE PREFIX（命名空间前缀） | BASE DIRECTORY（基目录） | RESULTING FILE PATH（最终的文件路径）     |
| -------------------------------------------- | -------------------------------- | ------------------------ | ----------------------------------------- |
| \Acme\Log\Writer\File_Writer                 | Acme\Log\Writer                  | ./acme-log-writer/lib/   | ./acme-log-writer/lib/File_Writer.php     |
| \Aura\Web\Response\Status                    | Aura\Web                         | /path/to/aura-web/src/   | /path/to/aura-web/src/Response/Status.php |
| \Symfony\Core\Request                        | Symfony\Core                     | ./vendor/Symfony/Core/   | ./vendor/Symfony/Core/Request.php         |
| \Zend\Acl                                    | Zend                             | /usr/includes/Zend/      | /usr/includes/Zend/Acl.php                |
