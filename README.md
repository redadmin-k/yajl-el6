# YAJL RPM for EL6

This repository provides an **unofficial EL6 RPM backport of YAJL (Yet Another JSON Library) 2.0.4**, based on the CentOS 7 `yajl-2.0.4-4.el7.src.rpm` source package.

The primary purpose is to provide the YAJL runtime library and development files required when building and using **ModSecurity v3 (libmodsecurity)** on Enterprise Linux 6.

## Overview

YAJL is a C library for parsing and generating JSON. ModSecurity v3 can use YAJL as a build dependency.

This repository is intended for **legacy-system compatibility testing, RPM builds, and Nginx WAF integration verification**. It is not an official CentOS or Red Hat package.

The RPMs are intended to provide:

- YAJL shared runtime library (`libyajl.so.2`)
- YAJL headers for compiling dependent software
- Development files, such as the unversioned linker library and pkg-config metadata, when included in the `yajl-devel` RPM

The RPMs use the normal system library and include paths rather than `/opt/modsecurity`. Actual installed file paths should be confirmed from the RPM manifest.

## Source and Backport

- **Upstream project:** [lloyd/yajl](https://github.com/lloyd/yajl)
- **Source distribution:** CentOS 7 `yajl-2.0.4-4.el7.src.rpm`
- **Target platform:** EL6 x86_64 (CentOS 6 / RHEL 6)
- **Purpose:** Supply YAJL dependencies for ModSecurity v3 on EL6

Original CentOS 7 source RPM:

```text
https://vault.centos.org/7.9.2009/os/Source/SPackages/yajl-2.0.4-4.el7.src.rpm
```

The source RPM is used as a packaging base and adapted for EL6. The resulting packages are **unofficial backports**, not EL6 packages distributed or supported by CentOS or Red Hat.

## Repository Layout

```text
README.md
SPEC/
RPMS/
Logs/
```

- `SPEC/` — RPM spec and packaging changes, if published
- `RPMS/` — Built RPM packages, if published
- `Logs/` — Build logs, if published

The contents of these directories depend on what has been committed to this repository.

## Install

Download the appropriate RPM packages from `RPMS/` and install both the runtime and development packages when building ModSecurity:

```bash
sudo yum localinstall RPMS/x86_64/yajl-2*.rpm RPMS/x86_64/yajl-devel-2*.rpm
```

If RPMs are stored directly under `RPMS/`, adjust the paths accordingly. Verify the selected filenames before installing, and avoid mixing EL6 and EL7 binary RPMs.

For runtime use only, `yajl` may be sufficient. Building ModSecurity generally also requires `yajl-devel`.

## Verify Installation

Check the installed packages:

```bash
rpm -qa | grep '^yajl'
```

Check their installed files:

```bash
rpm -ql yajl
rpm -ql yajl-devel
```

Check whether the dynamic linker can resolve YAJL:

```bash
ldconfig -p | grep libyajl
```

An example library entry is:

```text
libyajl.so.2 => /usr/lib64/libyajl.so.2
```

Actual paths and package release strings may vary with the RPM build.

## ModSecurity v3 Integration

This repository **does not install ModSecurity**. It supplies a library dependency that can be used in an EL6 build of ModSecurity v3.

Before building ModSecurity, install the development package and verify that the compiler can access the YAJL headers and libraries:

```bash
rpm -q yajl yajl-devel
ls /usr/include/yajl
ldconfig -p | grep libyajl
```

When using `rpmbuild` or `mock`, make the EL6 YAJL packages available in the build environment. A typical spec file may need:

```spec
BuildRequires: yajl-devel
```

A runtime dependency may be generated automatically from the linked shared library; inspect the final RPM with `rpm -qpR` to confirm the required dependencies.

If ModSecurity is installed under `/opt/modsecurity`, that installation prefix is independent of YAJL's system library paths.

## Build Notes

This backport is based on a **CentOS 7 SRPM**, not an EL6-native source package. Review any EL6-specific adjustments in the published spec and patches before rebuilding.

To rebuild an adapted source RPM on EL6, use:

```bash
rpmbuild --rebuild yajl-*.el6.src.rpm
```

This command assumes an EL6-adapted source RPM has already been created. Rebuilding the original `.el7.src.rpm` without changes is **not guaranteed** to work on EL6.

EL6's older compiler, CMake, and system libraries can require compatibility changes. Keep build logs and test results with the packaging materials when possible.

## Validation Scope

A successful RPM build does not by itself prove compatibility with ModSecurity or Nginx.

Recommended checks include:

1. Install `yajl` and `yajl-devel` on an EL6 test system.
2. Confirm the shared library is discoverable by `ldconfig`.
3. Build ModSecurity v3 against the EL6 YAJL development files.
4. Confirm ModSecurity's runtime linkage with `ldd`.
5. Test an Nginx build using the ModSecurity-nginx connector, if applicable.
6. Review ModSecurity audit logs and WAF behavior before any blocking-mode rollout.

The exact build and runtime validation results should be documented separately. This README does not claim that every EL6 configuration has been tested.

## Security and Production Notes

**CentOS 6 and RHEL 6 are legacy platforms.** This unofficial backport does not restore distribution security support or resolve vulnerabilities elsewhere in the operating system.

Use isolated test environments where possible. If legacy production use is unavoidable, assess network exposure, dependency vulnerabilities, and compensating controls independently.

YAJL is one dependency; it does not replace current ModSecurity, Nginx, OWASP CRS, or operating-system security maintenance.

## Disclaimer

This repository is unofficial.

It is not provided, maintained, endorsed, or supported by CentOS, Red Hat, the AlmaLinux OS Foundation, OWASP, or the upstream YAJL project.

The author provides no warranty of any kind. Use these packages at your own risk, and validate them carefully before deployment.

## License

YAJL source code remains subject to its upstream license and copyright notices. The CentOS source RPM may also contain packaging notices; preserve these as applicable.

Packaging changes in this repository are subject to the license statements included with the files. See the upstream project for source licensing details:

https://github.com/lloyd/yajl

