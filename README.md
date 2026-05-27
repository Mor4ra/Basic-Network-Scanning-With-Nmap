# Basic Network Scanning Using Nmap

## OverView
- This is a simple project showing how to use the tool `nmap` to scan a target to check for open ports & Services.
- For this specific project, I'll be scanning a local Kali VM from within itself.

## Setup
- First "dummy" services are enabled to test whether `nmap` catches them:
    ```
    sudo systemctl enable ssh
    sudo systemctl enable apache2
    ```
- Other Services can be enabled as we progress further.
- A simple scan is run and as mentioned, `nmap` picks the two services:
    ```
    $ nmap 127.0.0.1 # local host IP
    ...
    22/tcp  open ssh
    80/tcp  open http
    ```
- For this project, I'll be installing `vsftpd` as an extra service for `nmap` to scan. `vsftpd` is a service used by the FTP protocol for
file transfers, it's a common service and one may run into it while probing a network.
    ```
    sudo apt install vsftpd -y
    sudo systemctl start vsftpd # activates the service
    ```

## Scans
- Perfoming a basic scan shows the open services and their respective ports:
    ```
    $ nmap 127.0.0.1
    ...
    21/tcp  open    ftp
    22/tcp  open    ssh
    80/tcp  open    http
    ```
- A more detailed output can be found in the `nmap_scan_result.txt` file.

## Analysis
- Below is a breakdown of each port, it's purpose and what an attacker could deduct from it the moment they see it open:


**FTP/Port 21** - this is a protocol that enables file transfer between machines on a network, with it, one can download or upload files.
It's a legacy protocol and has since been replaced by SFTP, the secure version, FTP sends traffic in plain text which can be intercepted by an attacker
hence the need for SFTP.
- FTP can and has been exploited in the past, a misconfigured FTP server allows anonymous login which can be accessed without credentials, if
anonymous login is off, an attacker could try default or leaked credentials and if they get in they can download sensitive files and information or
upload malicious scripts.

**SSH/Port 22** - Secure Shell, usually called SSH is a protocol that gives you a secure, encrypted command line session on a remote machine. It's used
to remotely control and configure servers or access computers remotely. A malicious actor may try credential stuffing(trying username/password combos from
a leaked database) inorder to gain access to your machine/server remotely and if they succeed, they could set up backdoors to mantain persistence, an
example could be planting a cronjob that gives them adminstrator access each time a user logs in, Or simply add their SSH key to `~/.ssh/authorized_keys`
so they don't have to enter a password the next time they need access.

**HTTP/Port 80** - HTTP, HyperText Transfer Protocol, it's in the name, transfer hypertext(web content) from a server to a client(browser). the problem
with this protocol is that it's unencrypted, content served is done so plainly and can be intercepted using tools like `BurpSuite`. The modern replacement
is HTTPS, which is just Secure HTTP meaning web content is served while encrypted. Seeing this today either means it redirects you to Port 443(HTTPS)
or you're probing a misconfigured server. If it's a latter, this is juicy starting point to probe for vulnerabilities and exposed files, check for exposed
HTTP response headers which usually reveal the server name and version like `apache/2.4.1` which tells the attacker the exact kind of exploit to look for.
The attacker could run a directory brute-force tool like `gobuster` to find hidden directories like `/admin` that are linked anywhere.


## Conclusion
- `nmap` is a fundamental reconnaissance tool that reveals what services a target is running and whether they're open or not. Having this information
is crucial to understaning a system's attack surface - whether you're auditing your own infrastructure or doing a pentest on a client's network.
