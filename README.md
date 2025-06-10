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
    - name: Check memory
      shell: free -m | grep Mem | awk '{ print $4 }'
      register: free_mem
      tags: updates

    - name: Clear memory
      shell: sync; echo 1 > /proc/sys/vm/drop_caches
      when: free_mem.stdout | int < 384
      tags: updates

    - name: Recheck memory
      shell: free -m | grep Mem | awk '{ print $4 }'
      register: free_mem2
      tags: updates

    - name: Report systems without enough memory
      debug:
        msg: "WARNING: This system does not have enough memory to patch. Please reboot and try again."
      when: free_mem2.stdout | int < 384
      failed_when: free_mem2.stdout | int < 384
      tags: updates

    - name: Upgrade packages via yum with verbose output
      shell: |
        yum -v -y update > /home/user/patching_reports/yum_update_verbose.log 2>&1
      register: yum_update_result
      async: 1800
      poll: 60
      tags: updates

    - name: Print errors if yum failed
      debug:
        msg: "yum command produced errors"
      when: yumcommandout is not defined
      tags: updates

    - name: Make sure kernel installed
      tags:
      - updates
      - kernel

      block:
        - name: Check if new kernel was installed
          shell: /usr/bin/needs-restarting -r || /bin/true
          register: check_kernel
          changed_when: False

        - name: Find new initramfs version
          shell: "needs-restarting -r | grep kernel | awk '{ print $3 }'"
          register: initramfs
          when: "'kernel -> 3' in check_kernel.stdout"
          changed_when: False

        - name: Make sure new initramfs is present
          shell: "ls /boot/initramfs-{{ initramfs.stdout }}.x86_64.img"
          when: "'kernel -> 3' in check_kernel.stdout"
          changed_when: False

      rescue:
        - name: Recreate initramfs
          shell: dracut --kver `needs-restarting -r | grep kernel | awk '{ print $3 }'`.x86_64
