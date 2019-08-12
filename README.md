# Ansible Role: Installs Apache Tomcat Java Application server (optionally with Hugepages)

Installs Apache Tomcat Java Application server. Most complete Tomcat installation, supporting, init.d script, application naming, hugepages, hardening, beautiful error pages, sha512 hashed passwords, JMX configuration, multiple Tomcat versions, separated catalina_home and caralina_base.

[![Build Status](https://travis-ci.org/KAMI911/ansible-role-tomcat-multi.svg?branch=master)](https://travis-ci.org/KAMI911/ansible-role-tomcat-multi)

## Table of Contents

1. [Requirements][Requirements]
2. [Role Variables][Role Variables]
3. [Installation][Installation]
4. [Dependencies][Dependencies]
5. [Example Playbook][Example Playbook]
6. [Licensing][Licensing]
7. [Author Information][Author Information]
8. [Support][Support]
9. [Contributing][Contributing]
10. [Donation][Donation]

## Requirements

None.

## Installation

    ansible-galaxy install kami911.tomcat-multi

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    tomcat_majorversion: 8

Tomcat major version.

    tomcat_minorversion: 5

Tomcat minor version.

    tomcat_patchversion: 4

Tomcat micro version.

    tomcat_use_huge_pages: True

Use Huge Pages (Java calls it: UseLargePages) for enhance performance of Java applications. When a process uses some memory, the CPU is marking the RAM as used by that process. For efficiency, the CPU allocate RAM by chunks of 4K bytes (it's the default value on many platforms). Those chunks are named pages. Those pages can be swapped to disk, etc.

Since the process address space are virtual, the CPU and the operating system have to remember which page belong to which process, and where it is stored. Obviously, the more pages you have, the more time it takes to find where the memory is mapped. When a process uses 1GB of memory, that's 262144 entries to look up (1GB / 4K). If one Page Table Entry consume 8bytes, that's 2MB (262144 * 8) to look-up.

[Debian Wiki: Hugepages](https://wiki.debian.org/Hugepages)

When you enable it use [KAMI911:hugepages](https://galaxy.ansible.com/KAMI911/hugepages/) to configure Huge Pages in Linux.

    tomcat_file_encoding: UTF-8

Tomcat file encoding parameter: UTF-8

    tomcat_page_encoding: UTF-8

Tomcat page encoding parameter: UTF-8

    tomcat_access_log_pattern: "%h %l %u %t &quot;%r&quot; %s %b"

Pattern string of Tomcat access log. Tomcat [Access Logging](https://tomcat.apache.org/tomcat-9.0-doc/config/valve.html#Access_Logging):

Values for the pattern attribute are made up of literal text strings, combined with pattern identifiers prefixed by the "%" character to cause replacement by the corresponding variable value from the current request and response. The following pattern codes are supported:

    %a - Remote IP address
    %A - Local IP address
    %b - Bytes sent, excluding HTTP headers, or '-' if zero
    %B - Bytes sent, excluding HTTP headers
    %h - Remote host name (or IP address if enableLookups for the connector is false)
    %H - Request protocol
    %l - Remote logical username from identd (always returns '-')
    %m - Request method (GET, POST, etc.)
    %p - Local port on which this request was received. See also %{xxx}p below.
    %q - Query string (prepended with a '?' if it exists)
    %r - First line of the request (method and request URI)
    %s - HTTP status code of the response
    %S - User session ID
    %t - Date and time, in Common Log Format
    %u - Remote user that was authenticated (if any), else '-'
    %U - Requested URL path
    %v - Local server name
    %D - Time taken to process the request in millis. Note: In httpd %D is microseconds. Behaviour will be aligned to httpd in Tomcat 10 onwards.
    %T - Time taken to process the request, in seconds. Note: This value has millisecond resolution whereas in httpd it has second resolution. Behaviour will be align to httpd in Tomcat 10 onwards.
    %F - Time taken to commit the response, in millis
    %I - Current request thread name (can compare later with stacktraces)
    %X - Connection status when response is completed:
        X = Connection aborted before the response completed.
        + = Connection may be kept alive after the response is sent.
        - = Connection will be closed after the response is sent.

 There is also support to write information incoming or outgoing headers, cookies, session or request attributes and special timestamp formats. It is modeled after the Apache HTTP Server log configuration syntax. Each of them can be used multiple times with different xxx keys:

    %{xxx}i write value of incoming header with name xxx
    %{xxx}o write value of outgoing header with name xxx
    %{xxx}c write value of cookie with name xxx
    %{xxx}r write value of ServletRequest attribute with name xxx
    %{xxx}s write value of HttpSession attribute with name xxx
    %{xxx}p write local (server) port (xxx==local) or remote (client) port (xxx=remote)
    %{xxx}t write timestamp at the end of the request formatted using the enhanced SimpleDateFormat pattern xxx

All formats supported by SimpleDateFormat are allowed in %{xxx}t. In addition the following extensions have been added:

    sec - number of seconds since the epoch
    msec - number of milliseconds since the epoch
    msec_frac - millisecond fraction

These formats cannot be mixed with SimpleDateFormat formats in the same format token.

Furthermore one can define whether to log the timestamp for the request start time or the response finish time:

    begin or prefix begin: chooses the request start time
    end or prefix end: chooses the response finish time

By adding multiple %{xxx}t tokens to the pattern, one can also log both timestamps.

The shorthand pattern pattern="common" corresponds to the Common Log Format defined by '%h %l %u %t "%r" %s %b'.

The shorthand pattern pattern="combined" appends the values of the Referer and User-Agent headers, each in double quotes, to the common pattern.

    tomcat_use_secure_flag: True

Set this attribute to True if you wish to have calls to request.isSecure() to return true for requests received by this Connector. You would want this on an SSL Connector or a non SSL connector that is receiving data from a SSL accelerator, like a crypto card, a SSL appliance or even a webserver. The default value is False.

    tomcat_session_http_only: True

Forcing Tomcat to use JSESSIONID cookie over only http.

    tomcat_session_secure: True

Forcing Tomcat to use secure JSESSIONID cookie.

    tomcat_manage_java_pkg: False

Tomcat manage java installation an install OpenJDK or not.

    tomcat_system_name: "tomcat_app"

Use this folder name for this tomcat main folder.

    tomcat_system_home: "{{ tomcat_base_folder }}/{{ tomcat_system_user }}"

Folder of Tomcat binaries using tomcat_system_home variable.

    tomcat_catalina_home: '{{ tomcat_system_home }}/tomcat{{ tomcat_majorversion }}'

Tomcat Cataline home folder.

    tomcat_instance:

List of dictioneries where the tomcat instances are configured.

      - instance_name: 'tomcat_app_sys1'

Name of first instance. Also this variable used as name of instance folder name, start/stop script and process identifier.

        http_port: 8080

Port number of http port of Tomcat instance sevice. Please define it carefuly, it should be not the same as other ports.

        http_stp: false

Tomcat "Connector" using the shared thread pool for http connections.

        http_zone: "internal"

The firewalld zone name where connection are accepted for http connections. This variable is used when firewalld supported system is used (for exmple: CentOS 7) and tomcat_manage_firewalld_use_zone variable is true.

        http_source:  # Tweak this according yout network
          - "0.0.0.0/0"

List of source ports where connection are accepted for http connections. This time only firewalld is supported. The default values are 0.0.0.0/0 means all connection is accepted. This should narrowed.

        ajp_port: 8009

Port number of ajp port of Tomcat instance sevice. Please define it carefuly, it should be not the same as other ports.

        ajp_stp: false

Tomcat "Connector" using the shared thread pool for ajp connections.

        ajp_zone: "trusted"

The firewalld zone name where connection are accepted for ajp connections. This variable is used when firewalld supported system is used (for exmple: CentOS 7) and tomcat_manage_firewalld_use_zone variable is true.

        ajp_source:  # Tweak this according yout network
          - "0.0.0.0/0"

List of source ports where connection are accepted for ajp connections. This time only firewalld is supported. The default values are 0.0.0.0/0 means all connection is accepted. This should narrowed.

        https_port: 8443

Port number of https port of Tomcat instance sevice. Please define it carefuly, it should be not the same as other ports.

        https_zone: "internal"

The firewalld zone name where connection are accepted for https connections. This variable is used when firewalld supported system is used (for exmple: CentOS 7) and tomcat_manage_firewalld_use_zone variable is true.

        https_source:  # Tweak this according yout network
          - "0.0.0.0/0"

List of source ports where connection are accepted for https connections. This time only firewalld is supported. The default values are 0.0.0.0/0 means all connection is accepted. This should narrowed.

        jmx_port: 48080

Port number of JMX manager port of Tomcat instance sevice. Please define it carefuly, it should be not the same as other ports.

        jmx_zone: "internal"

The firewalld zone name where connection are accepted for JMX manager connections. This variable is used when firewalld supported system is used (for exmple: CentOS 7) and tomcat_manage_firewalld_use_zone variable is true.

        jmx_source:  # Tweak this according yout network
          - "0.0.0.0/0"

List of source ports where connection are accepted for JMX manager connections. This time only firewalld is supported. The default values are 0.0.0.0/0 means all connection is accepted. This should narrowed.

        shutdown_port: 49080

Port number of JMX shutdown port of Tomcat instance sevice. Please define it carefuly, it should be not the same as other ports.

        shutdown_zone: "internal"

The firewalld zone name where connection are accepted for JMX shutdown connections. This variable is used when firewalld supported system is used (for exmple: CentOS 7) and tomcat_manage_firewalld_use_zone variable is true.

        shutdown_source:  # Tweak this according yout network
          - "127.0.0.1/32"

List of source ports where connection are accepted for JMX shutdown connections. This time only firewalld is supported. The default values are "127.0.0.1/32 means only local connection is accepted. This should narrowed.

        juli_logging_level: "FINE"

Set [1catalina|2localhost|3manager|4host-manager].org.apache.juli.AsyncFileHandler.level and java.util.logging.ConsoleHandler.level to this loglevel. Possible values are:
  SEVERE, WARNING, INFO, CONFIG, FINE, FINER, FINEST or ALL. Default is FINE.

    tomcat_manage_firewalld: true

Role manages the firewalld settings of required ports.

    tomcat_manage_firewalld_use_zone: true

Tomcat firewalld uses zones (default) or use source addresses.

    tomcat_catalina_logs_directory_mode: "u=rwx,g=rwx,o="

Tomcat catalina logs directory mode.

    tomcat_java_version: 11

Configure Tomcat to use the specified version version of Java.

Tomcat LDAP authentication configuration:

    tomcat_ldap_enable: false

Enable Tomcat LDAP authentication. Disabled by default.

    tomcat_ldap_debug_level: 99

Tomcat LDAP authentication debug level. Default is 99.

    tomcat_ldap_url: 'ldap://ldap.cloud.department.ca:389'

Tomcat LDAP authentication URL. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_user: 'technicaluser@cloud.department.ca'

Tomcat LDAP user to reach LDAP server. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_pass: 'password'

Tomcat LDAP user's password to reach LDAP server. Do not use the default value. Tweak this value according your settings.

    tomcat_ldap_user_ou: 'ou=Users,dc=cloud,dc=department,dc=ca'

Organization Unit of valid users for Tomcat LDAP authentication. Tweak this value according your settings.

    tomcat_ldap_user_name: "(sAMAccountName={0})"

Name of authenticated users for Tomcat LDAP authentication. Default setting is (sAMAccountName={0}) which perfect for Windows Active Directory.

    tomcat_ldap_user_referrals: 'follow'



    tomcat_ldap_user_subtree: true

Tomcat LDAP user could be on the subtree of Organization Unit.

    tomcat_ldap_role_ou: 'ou=TomcatAdmin,ou=Groups,dc=cloud,dc=department,dc=ca'

Organization Unit of group of users for Tomcat LDAP authentication. Tweak this value according your settings. Create security groups here with the Tomcat role names like "manager-gui".

    tomcat_ldap_role_name: 'name'

User security group name as Tomcat role names like "manager-gui". Default is good if you want to use Tomcat role names.

    tomcat_ldap_role_subtree: true

Tomcat LDAP group could be on the subtree of Organization Unit.

    tomcat_ldap_role_search: '(member={0})'

Every member matching the security group's name could access the server as specified in the role.

    debug_enable: false

Enable Tomcat debug port.

    debug_port: 8000

Specify Tomcat debug port.

    debug_sever: {{ ansible_hostname }} # or 127.0.0.1

Permit connection from all location of from local connection only.

    debug_parameter: '-agentlib:jdwp=transport=dt_socket,address={{ item.debug_sever }}:{{ tomcat_debug_port }},server=y,suspend=n'

Specify Tomcat debug parameters.

## Dependencies

None.

## Example Playbook

    - hosts: all
      roles:
        - tomcat-multi

## Licensing

The lactransformer application and documantations are licensed under the terms of
the MIT / BSD, you will find a copy of this license in the
[LICENSE](LICENSE) file included in the source package.

## Author Information

This role was created in 2016-2019 by Kálmán Szalai - KAMI

## Support

If you have any question, do not hesitate and drop me a line.
If you found a bug, or have a feature request, you can [fill an issue](https://github.com/KAMI911/ansible-role-tomcat-multi/issues).

## Contributing

There are many ways to contribute to ansible-role-tomcat-multi -- whether it be sending patches,
testing, reporting bugs, or reviewing and updating the documentation. Every
contribution is appreciated!

Please continue reading in the [contributing chapter](CONTRIBUTING.md).

### Fork me on Github

https://github.com/KAMI911/ansible-role-tomcat-multi

Add a new remote `upstream` with this repository as value.

```
git remote add upstream https://github.com/KAMI911/ansible-role-tomcat-multi.git
```

You can pull updates to your fork's master branch:

```
git fetch --all
git pull upstream HEAD
```

## Donation

If you find this useful, please consider a donation:

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=RLQZ58B26XSLA)

<!-- TOC URLs -->
[Requirements]: #requirements
[Role Variables]: #role_variables
[Installation]: #installation
[Dependencies]: #dependencies
[Example Playbook]: #example_playbook
[Licensing]: #licensing
[Author Information]: #author_information
[Support]: #support
[Contributing]: #contributing
[Donation]: #donation
