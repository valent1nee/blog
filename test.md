# A Journey Into Finding Vulnerabilities in the PMB Library Management System

## Introduction
The following research was conducted by Nexa’s security team with the goal to contribute to the ongoing efforts to improve the security of open-source software and protect the organizations that rely on them.

In the search for open source projects to look for vulnerabilities, the idea of reviewing the code of a library system called PMB used by several well-known organizations came up. PMB, being an open-source project, provides an excellent opportunity for security researchers to study its codebase, identify potential vulnerabilities, and contribute to its security.

According to Wikipedia, "PMB is a fully featured open source integrated library system. It is continuously developed and maintained by the French company PMB Services."

For instance, using the following Google Dork, almost 5 million results were obtained, which indicates the widespread use of this software.

![image](https://github.com/user-attachments/assets/397f56ba-3eca-42bc-afc8-832c9affa341)
Google search for PMB

## Plan
The objective was to find critical impact vulnerabilities that would allow bypassing authentication and subsequently achieve remote command execution using authenticated components.

![image](https://github.com/user-attachments/assets/18f57fc6-b9c3-4b64-89f2-0259e2a3e78b)
Authenticated and unauthenticated components

## Setup
In order to conduct the research, a local environment composed of the following was prepared:
 - Ubuntu 22.04 LTS instance.
 - PHP modules required to use the software (PMB).
 - A database server (MariaDB) with logging enabled to analyze the SQL queries performed by the application.
 - Xdebug and Visual Studio Code to debug the application.

## Results

The summary of the identified vulnerabilities is listed below (for PMB version 7.4.7 and earlier).

**Pre-authenticated vulnerabilities**

- CVE-2023-52153: SQL injection in /pmb/opac_css/includes/sessions.inc.php
 - CVSS: 7.5 (High)
- CVE-2023-37177: SQL injection in /admin/convert/export_z3950.php
 - CVSS: 7.5 (High)
- CVE-2023-51828: SQL injection in /admin/convert/export_z3950_new.php
 - CVSS: 7.5 (High)
- CVE-2023-38844: SQL injection in export_skos.php
 - CVSS: 7.5 (High)

**Post-authenticated vulnerabilities**

- Insecure file upload in /admin/convert/start_import.php
 - CVSS: 7.2 (High)
- CVE-2023-52154: Insecure file upload in camera_upload.php
 - CVSS: 7.2 (High)
- CVE-2023-52155: SQL injection to RCE in /admin/sauvegarde/run.php
 - CVSS: 8.8 (High)
