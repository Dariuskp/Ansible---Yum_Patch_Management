# Ansible Yum Patch Management

This repository contains an Ansible playbook designed to automate the patch management process on RHEL-based Linux servers using `yum`. The playbook installs updates, manages repositories, performs memory checks, generates update reports, and sends email notifications to system administrators.

---

## Overview

Efficient patch management is critical for maintaining system security and stability. This playbook simplifies patching by automating:

- Checking and updating system packages via `yum`
- Managing repository configurations
- Monitoring system memory status before applying patches
- Generating detailed patch reports
- Sending email notifications with update summaries and logs

---

## Features

- Installs all available package updates
- Verifies and updates YUM repositories configuration
- Checks free memory before patching to avoid out-of-memory issues
- Generates update reports in `/home/dariusp/patching_reports/`
- Sends email notifications with patching results and details
- Fully automated with no manual intervention needed during runs

---

## Prerequisites

- Ansible installed on the control machine
- SSH access to target RHEL/CentOS/Rocky Linux hosts
- Target hosts configured with `yum` package manager
- `mailx` or compatible mail client installed on target hosts for email notifications
- Proper email configuration on target hosts to send mail (SMTP setup or MTA)

---

## Playbook Breakdown

### Variables

- `patching_reports_dir`: Directory path where patch reports will be stored (e.g., `/home/dariusp/patching_reports/`)
- Email recipient(s) and sender configured within the playbook or external vars

### Key Tasks

```yaml

