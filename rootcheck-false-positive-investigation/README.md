# Wazuh Rootcheck Alert Investigation

## 1. Objective

The objective of this activity is to investigate a Rootcheck alert detected by Wazuh on the Linux machine. The alert identified `/usr/bin/md5sum` as a potentially trojaned file. The investigation focuses on checking the flagged file, identifying its actual location, checking its package ownership, and verifying whether there are any signs of file modification.

## 2. Lab Environment

| Component | Details |
|---|---|
| SIEM | Wazuh |
| Operating System | Ubuntu Linux |
| Hostname | Linux-PC |
| IP Address | 192.168.1.13 |
| Wazuh Agent | Linux_Agent |
| Investigation Tools | Wazuh Dashboard and Linux Terminal |

## 3. Alert Detection

While checking the events in Wazuh Dashboard, I found a Rootcheck alert from the Linux agent.

**Alert Information:**
- Rule ID: 510
- Description: Host-based anomaly detection event (rootcheck)
- Severity Level: 7
- Detection Source: Rootcheck
- Flagged File: `/usr/bin/md5sum`
- Detection: Trojaned version of file detected

The alert indicated that the file matched a generic suspicious signature. Because of this, I decided to investigate the file before considering it a possible malware infection.

## 4. Investigation and Evidence

### 4.1 Checking the Flagged File

First, I checked the file information using:

```bash
ls -l /usr/bin/md5sum
```

The output showed that `/usr/bin/md5sum` is a symbolic link pointing to another file under the Cargo Coreutils directory.

### 4.2 Checking the Actual File Location

To identify the actual location of the file, I used:

```bash
readlink -f /usr/bin/md5sum
```

The result showed:

```text
/usr/lib/cargo/bin/coreutils/md5sum
```

This confirmed that the flagged file was linked to the Rust Coreutils installation.

### 4.3 Identifying the File Type

Next, I checked the actual binary using:

```bash
file /usr/lib/cargo/bin/coreutils/md5sum
```

The output identified it as a 64-bit ELF executable for Linux.

### 4.4 Checking Package Ownership

I used the following command to determine which installed package owns the file:

```bash
dpkg -S /usr/lib/cargo/bin/coreutils/md5sum
```

The result showed:

```text
rust-coreutils: /usr/lib/cargo/bin/coreutils/md5sum
```

This means the file belongs to the `rust-coreutils` package.

### 4.5 Verifying Package Integrity

To check whether the package files had any reported discrepancies, I executed:

```bash
sudo dpkg -V rust-coreutils
```

The command returned no output. This means the package verification process did not report any discrepancies.

### 4.6 Checking Package Repository Information

I also checked the package information using:

```bash
apt-cache policy rust-coreutils
```

The output showed the installed version and a newer candidate version available from the configured Ubuntu archive.

This provided additional information about the package available through the system's configured repository.

### 4.7 Checking Package Installation History

Lastly, I checked the current package log for installation records:

```bash
grep -i "rust-coreutils" /var/log/dpkg.log
```

No matching entries were found in the current log. Therefore, I could not determine the package's installation date using this command.

## 5. Investigation and Analysis

Based on the investigation, I found that the flagged `/usr/bin/md5sum` file was not a standalone binary but a symbolic link pointing to the Rust Coreutils installation.

I also confirmed that the actual binary belongs to the `rust-coreutils` package. When I performed package verification, no discrepancies were reported.

These findings suggest that the alert may have been triggered because of the alternative Coreutils implementation installed on the Linux machine.

However, the investigation did not fully verify the original package source or the exact reason why Rootcheck matched the file with its generic suspicious signature.

## 6. Findings and Conclusion

The Wazuh Rootcheck alert identified `/usr/bin/md5sum` as a potentially trojaned file. After checking the file and its package information, I found that it points to a binary belonging to the installed `rust-coreutils` package.

The package verification also returned no reported discrepancies.

Based on the available evidence, the alert is likely a false positive. However, this cannot be confirmed completely because further verification of the original package source and the Rootcheck signature is still needed.

No confirmed malware infection was identified during this activity.

## 7. Recommended Response

If this alert occurred in an actual SOC environment, I would:

1. Record the alert and the affected Linux endpoint.
2. Collect and preserve the file information and investigation results.
3. Document the possibility of a false positive.
4. Avoid deleting or modifying the flagged file without authorization.
5. Escalate the alert for further validation if required by the organization's procedures.

## 8. Lessons Learned

- Not all security alerts indicate an actual attack or malware infection.
- Rootcheck can generate false-positive detections.
- Checking file locations and package ownership helps identify suspicious files.
- Package verification can provide useful information about possible file modifications.
- Alerts should be investigated and supported by evidence before making a conclusion.
- Proper documentation is important when handling and reporting security alerts.
