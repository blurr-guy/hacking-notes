# Web Application Security Fundamentals

> TryHackMe module Walkthrough - Personal Notes

---

## Table of Contents
- [Walking An Application](#walking-an-application)
- [Content Discovery](#content-discovery)
- [Web Server Attacks - I](#web-server-attacks---i)
- [Web Server Attacks - II](#web-server-attacks---ii)

---

## Walking An Application

Walking an application is important to understand its functionality and features. It will increase the attack surface for us.

### Viewing Source Code

One of the most important techniques to understand the application. You can even get clues and potentially leaked information from site comments.

Use `view-source:https://domain.name/` to view the source code of any site.

---

## Content Discovery

### Important Files
- **robots.txt** - Contains crawling restrictions
- **sitemap.xml** - Can potentially leak input endpoints to the application

### Reconnaissance Techniques

**Response Headers Analysis**
- Manually check headers of the response given by the server
- You will find information about the server and language used

**Google Dorking**
- Try different Google dorking searches about the site and what you want to find
- Use [Google Advanced Search](https://www.google.com/advanced_search) for advanced searching

**Technology Detection**
- Use Wappalyzer to find technologies used by the site

**WayBack Machine**
- Repository of the old internet
- Has snapshots of webpages and URLs

**GitHub**
- Modern SaaS and sites have codebase stored here

**S3 Buckets**
- Provides simple storage services
- Check for misconfigurations
- URL format: `https://{name}.s3.amazonaws.com`

### Directory and Subdomain Enumeration

**Directory/File Listing**
- Use tools like `gobuster` and `FFUF` for listing directories and files of a site

**Subdomain Enumeration**
- To increase attack surface on a site, check how many subdomains it has
- Use DNS enum mode of `gobuster` and `FFUF`

---

## Web Server Attacks - I

## Web Server Attacks - I

### Identifying Web Server

**Using curl to identify servers:**

```bash
curl -sI url          # -I option to get only headers of the response
curl -s url/non-available-path  # Error messages show which server is running
```

Error messages and format can tell us which server is behind it.

### Python Server

- Serves the current directory in which it is opened

### Apache Server

**Status checking:**
- `/server-status` - Shows the status of the server and more information
- Use directory listing to list all the files of the server

### Node.js Server

**Identifying Node.js:**
- Look for `X-powered-by: Express` header in response

**Characteristics:**
- Behave differently from Apache and Python servers
- Not serving static files from a configured document root
- Running application code that decides what to return for every request

**Route Discovery:**
```bash
curl -s http://10.146.141.103:3000/api/routes  # Lists all routes of the application
```

**Static Files:**
- Node.js serves static files from the `/static` directory

### Nginx Server

**Directory Listing:**
- Nginx does not enable directory listing by default
- When exposed, developers add `autoindex` to a location block in configuration

**Configuration Example:**
```nginx
location /nginx_status {
    stub_status;
    allow all;  # Should be: allow 127.0.0.1; deny all;
}
```

**Checking Nginx Status:**
```bash
curl -s http://10.146.141.103:8080/nginx_status
```

**Configuration Files:**
- Located in `/etc/nginx/` on Ubuntu
- Site configuration in `/etc/nginx/sites-available/` shows exactly what directories are exposed and what modules are enabled

---

## Web Server Attacks - II

## Web Server Attacks - II

### IIS Server

IIS is a Windows-based server which hosts applications.

### IIS Server Architecture

**HTTP.sys**
- In IIS 6+, TCP/IP and HTTP parsing happen in a kernel-mode driver called HTTP.sys (not in a userland process)

**WAS (Windows Process Activation Service)**
- Sits above HTTP.sys in userland
- Reads `applicationHost.config` (the master IIS config, XML-based since IIS 7)
- Manages application pool configuration
- Responsible for starting/stopping worker processes on demand
- Handles non-HTTP protocol activation (net.tcp, net.pipe for WCF)

**WWW Service (W3SVC)**
- The userland Windows service that works with WAS
- Manages HTTP.sys configuration
- Pushes routing rules, SSL bindings, etc. down into the kernel driver

**Application Pools and Worker Processes (w3wp.exe)**
- Each application pool is a configuration + isolation boundary
- Sites/applications get assigned to a pool

### Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                          CLIENT REQUEST                                │
│                    (HTTP/HTTPS on port 80/443)                         │
└───────────────────────────────┬──────────────────────────────────────┘
                                 │
                                 ▼
╔══════════════════════════════════════════════════════════════════════╗
║  KERNEL MODE                                                           ║
║  ┌────────────────────────────────────────────────────────────────┐  ║
║  │  HTTP.sys (kernel driver)                                        │  ║
║  │  - Owns TCP/port binding (shared across all sites)                │  ║
║  │  - Parses raw HTTP                                                │  ║
║  │  - Kernel-mode response cache                                     │  ║
║  │  - Routes by Host header → correct request queue                  │  ║
║  │  - No match / pool down → returns 503 directly                    │  ║
║  └───────────────────────────┬────────────────────────────────────┘  ║
╚══════════════════════════════╪═════════════════════════════════════╝
                                │  (request queue handoff)
                                ▼
╔══════════════════════════════════════════════════════════════════════╗
║  USER MODE                                                              ║
║                                                                          ║
║  ┌───────────────┐        ┌───────────────────────────────────────┐   ║
║  │  WWW Service   │◄──────►│  WAS (Windows Process Activation Svc)  │   ║
║  │  (W3SVC)       │        │  - Reads applicationHost.config        │   ║
║  │  pushes config │        │  - Starts/stops worker processes       │   ║
║  │  to HTTP.sys   │        │  - Manages app pool lifecycle           │   ║
║  └───────────────┘        └───────────────┬───────────────────────┘   ║
║                                            │                            ║
║                                            ▼                            ║
║              ┌─────────────────────────────────────────────┐          ║
║              │         APPLICATION POOLS (isolation)         │          ║
║              │                                                │          ║
║              │  ┌───────────────┐   ┌───────────────┐        │          ║
║              │  │ Pool: SiteA    │   │ Pool: SiteB    │  ...   │          ║
║              │  │ Identity:      │   │ Identity:      │        │          ║
║              │  │ AppPoolIdentity│   │ Custom/NetSvc  │        │          ║
║              │  └───────┬───────┘   └───────┬───────┘        │          ║
║              └──────────┼───────────────────┼────────────────┘          ║
║                         ▼                   ▼                            ║
║              ┌──────────────────┐ ┌──────────────────┐                 ║
║              │  w3wp.exe         │ │  w3wp.exe         │                 ║
║              │  (worker process) │ │  (worker process) │                 ║
║              │                    │ │                    │                 ║
║              │  ┌──────────────┐ │ │  ┌──────────────┐ │                 ║
║              │  │ INTEGRATED   │ │ │  │ INTEGRATED   │ │                 ║
║              │  │ PIPELINE     │ │ │  │ PIPELINE     │ │                 ║
║              │  │              │ │ │  │              │ │                 ║
║              │  │ BeginRequest │ │ │  │  ...same     │ │                 ║
║              │  │ Authenticate │ │ │  │  stages      │ │                 ║
║              │  │ Authorize    │ │ │  │              │ │                 ║
║              │  │ ResolveCache │ │ │  │              │ │                 ║
║              │  │ MapHandler   │ │ │  │              │ │                 ║
║              │  │ ExecuteHandler│ │ │  │              │ │                 ║
║              │  │ ReleaseState │ │ │  │              │ │                 ║
║              │  │ EndRequest   │ │ │  │              │ │                 ║
║              │  │              │ │ │  │              │ │                 ║
║              │  │ Native +     │ │ │  │              │ │                 ║
║              │  │ Managed      │ │ │  │              │ │                 ║
║              │  │ modules fire │ │ │  │              │ │                 ║
║              │  │ on EVERY     │ │ │  │              │ │                 ║
║              │  │ request      │ │ │  │              │ │                 ║
║              │  └──────────────┘ │ │  └──────────────┘ │                 ║
║              └──────────────────┘ └──────────────────┘                 ║
╚══════════════════════════════════════════════════════════════════════╝
                                │
                                ▼
        ┌───────────────────────────────────────────────────┐
        │  CONFIG HIERARCHY (inherited, overridable)          │
        │                                                       │
        │  applicationHost.config  (machine/site-wide)          │
        │            │                                          │
        │            ▼                                          │
        │  web.config (site root)                                │
        │            │                                          │
        │            ▼                                          │
        │  web.config (app/subfolder — overrides unless          │
        │              parent sets allowOverride="false")        │
        └───────────────────────────────────────────────────┘
```

### Fingerprinting the IIS Server

Use standard curl command:
```bash
curl -I url
```

### WebDAV Detection with OPTIONS

**Web Distributed Authoring and Versioning (WebDAV)** is an HTTP extension that adds file management verbs:
- PUT (upload)
- DELETE
- COPY
- MOVE
- PROPFIND
- LOCK

Legitimate use cases include SharePoint and web-based file editing tools. When left enabled on a directory with write and script execute permissions, it becomes a direct path to uploading an executable shell.

**Detection:**
```bash
curl -X OPTIONS url 2>&1 | grep -E "Allow:|DAV:"
```

**Exploitation Example:**
```bash
curl -s -o /dev/null -w "PUT aspx: %{http_code}\n" -X PUT --data '<%@ Page Language=Jscript%><%Response.Write(1+1)%>' http://10.146.168.87/webdav/test.aspx
```

### Tilde Enumeration

The tilde enumeration technique leaks the names of files and directories that standard browsing and directory brute-forcing would never find.

**How it works:**
- All NTFS files follow the 8.3 naming pattern (8 char name and 3 char file extension)
- The conversion rule is predictable:
  1. Take the first 6 characters of the long name
  2. Append `~1` (or `~2`, `~3` if there is a collision)
  3. Keep the first 3 characters of the extension

Use custom script for enumeration.

### WebDAV Exploitation

**Prerequisites - All three must be true simultaneously:**

1. WebDAV is enabled on the target directory
2. Valid credentials with Write permission on the WebDAV directory
3. Script Execute is set (IIS passes .aspx requests to the ASP.NET handler rather than serving them as static content)

**ASPX Shells:**
- An ASPX web shell is an ASP.NET file hosted on a web server that accepts attacker input via HTTP and executes it under the server process
- From the server's perspective, it is just another .aspx page
- From the attacker's perspective, it is a remote command interface
- Getting an ASPX shell means remote command execution on the host

### IIS Misconfigurations

#### Directory Listing Enabled
- With directory listing, we can map out the application and find critical endpoints

#### HTTP PUT and DELETE Without Authentication
- Attacker can use these HTTP methods to delete web content and upload files (reverse shells) without authentication

#### web.config Exposure
- `web.config` is the ASP.NET application configuration file
- Contains database connection strings, API keys, SMTP credentials, encryption keys, and application settings
- IIS normally blocks `.config` files via request filtering rule or handler mapping
- If that rule is removed or a MIME mapping is added incorrectly, `web.config` becomes downloadable

#### Verbose Error Messages
- IIS in development mode returns full .NET stack traces when an application error occurs
- Exposes internal file paths like `C:\inetpub\wwwroot\App\Controllers\AccountController.cs`
- Reveals .NET framework version, database query that failed, and sometimes the server's internal IP address
- In production, the `customErrors` setting in `web.config` should be set to `On` to replace stack traces with a generic error page

#### trace.axd Left Enabled
- ASP.NET includes a built-in diagnostic handler at `trace.axd`
- When enabled, visiting `http://target/trace.axd` returns the application trace log for recent requests
- Log includes headers, form values, session state, cookies, and internal timing data for every recent request
- By default, holds the 50 most recent requests (`requestLimit="50"` in the `<trace>` element)
- On active servers, that window closes quickly

#### TRACE Method Enabled
- The HTTP TRACE method is different from the ASP.NET `trace.axd` handler
- Echoes the incoming request back to the caller
- Designed for diagnostic loopback testing
- In production, serves no legitimate purpose
- Can enable Cross-Site Tracing (XST) attacks
- Correct state is 405 Method Not Allowed (not 200 with echo)

#### Application Pool Running as a Privileged Account
- Default AppPool identity (ApplicationPoolIdentity) is low-privilege
- Administrators sometimes configure AppPools to run as SYSTEM, Administrator, or domain admin account
- Often done to avoid permission errors on file shares or databases
- This is a significant security risk

### Automation of Above Workflow

Nmap has many scripts to automate these tasks:

**Service Version Detection:**
```bash
nmap -sV -p 80 target_ip
```

**Enumerating HTTP Methods:**
```bash
nmap --script http-methods -p 80 ip_of_target
```

**Detecting WebDAV:**
```bash
nmap --script http-webdav-scan -p 80 target_ip
```
The `http-webdav-scan` script probes for WebDAV by sending a PROPFIND request and reading the DAV response headers.

**Identifying Authentication Requirements:**
```bash
nmap --script http-ntlm-info --script-args http-ntlm-info.root=/webdav/ -p 80 target-ip
```

---

**End of Notes**





























