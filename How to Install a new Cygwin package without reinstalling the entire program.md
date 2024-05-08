---
title: How to Install a new Cygwin Package Without Reinstalling the Entire Program
layout: note
tags:
  - Cygwin
  - Windows
---

- Download the Cygwin setup program and put it in your home directory for easy execution.
- In Windows, find where Cygwin binary is located on disk and **run as admin**

`./setup-x86_64.exe --root e:\cygwin64 --no-desktop --no-shortcuts --no-startmenu --quiet-mode -P wget`

You can run `./setup-x86_64.exe -h` to get a list of options.
You can also update with this command:
``./setup-x86_64.exe --root e:\cygwin64 -q -g`

Check the console output log. should be like:

```
Starting cygwin install, version 2.924
User has backup/restore rights
User has symlink creation right
Current Directory: C:\Users\benfr\Downloads
root: E:\cygwin64\home\benfr\cygwin64 system
Selected local directory: C:\Users\benfr\Downloads
net: Preconfig
site: https://cygwin.mirror.constant.com/
solving: 1 tasks, update: no, use test packages: no
solving: 38 tasks, update: no, use test packages: no
Augmented Transaction List:
   0 install libbrotlicommon1        1.0.9-2
   1 install libcares2               1.14.0-1
   2 install libcom_err2             1.44.5-1
   3 install libcrypt2               4.4.20-1
   4 install libdb5.3                5.3.28-2
   5 install libgpg-error0           1.37-1
   6 install libgsasl-common         2.2.0-2
   7 install libidn12                1.41-1
   8 install libidn2_0               2.3.4-1
   9 install libkrb5support0         1.15.2-2
  10 install libnettle6              3.4.1-1
  11 install libnghttp2_14           1.51.0-1
  12 install libntlm0                1.4-1
  13 install libssh2_1               1.10.0-1
  14 install libunistring2           0.9.10-1
  15 install libunistring5           1.1-1
  16 install libusb0                 1.2.6.0-2
  17 install libxml2                 2.9.12-2
  18 install libzstd1                1.5.2-1
  19 install publicsuffix-list-dafsa 20221115-1
  20 install libbrotlidec1           1.0.9-2
  21 install libgcrypt20             1.10.1-1
  22 install libassuan0              2.5.5-1
  23 install libk5crypto3            1.15.2-2
  24 install libhogweed4             3.4.1-1
  25 install libmetalink3            0.1.3-1
  26 install libpsl5                 0.21.1-2
  27 install libkrb5_3               1.15.2-2
  28 install libgnutls30             3.6.9-1
  29 install libgssapi_krb5_2        1.15.2-2
  30 install libopenldap2_4_2        2.6.3-1
  31 install libsasl2_3              2.1.27-1
  32 install libopenldap2            2.6.3-1
  33 install libgsasl18              2.2.0-2
  34 install libcurl4                7.86.0-2
  35 install gnupg                   1.4.23-1
  36 install libgpgme11              1.9.0-1
  37 install wget                    1.21.3-1
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/brotli/libbrotlicommon1/libbrotlicommon1-1.0.9-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/c-ares/libcares2/libcares2-1.14.0-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/e2fsprogs/libcom_err2/libcom_err2-1.44.5-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libxcrypt/libcrypt2/libcrypt2-4.4.20-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/db/libdb5.3/libdb5.3-5.3.28-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libgpg-error/libgpg-error0/libgpg-error0-1.37-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/gsasl/libgsasl-common/libgsasl-common-2.2.0-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libidn/libidn12/libidn12-1.41-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libidn2/libidn2_0/libidn2_0-2.3.4-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/krb5/libkrb5support0/libkrb5support0-1.15.2-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/nettle/libnettle6/libnettle6-3.4.1-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/nghttp2/libnghttp2_14/libnghttp2_14-1.51.0-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libntlm/libntlm0/libntlm0-1.4-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libssh2/libssh2_1/libssh2_1-1.10.0-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libunistring/libunistring2/libunistring2-0.9.10-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libunistring/libunistring5/libunistring5-1.1-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libusb-win32/libusb0/libusb0-1.2.6.0-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libxml2/libxml2-2.9.12-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/zstd/libzstd1/libzstd1-1.5.2-1.tar.zst
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
noarch/release/publicsuffix-list/publicsuffix-list-dafsa/publicsuffix-list-dafsa
-20221115-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/brotli/libbrotlidec1/libbrotlidec1-1.0.9-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libgcrypt/libgcrypt20/libgcrypt20-1.10.1-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libassuan/libassuan0/libassuan0-2.5.5-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/krb5/libk5crypto3/libk5crypto3-1.15.2-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/nettle/libhogweed4/libhogweed4-3.4.1-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libmetalink/libmetalink3/libmetalink3-0.1.3-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/libpsl/libpsl5/libpsl5-0.21.1-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/krb5/libkrb5_3/libkrb5_3-1.15.2-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/gnutls/libgnutls30/libgnutls30-3.6.9-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/krb5/libgssapi_krb5_2/libgssapi_krb5_2-1.15.2-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/openldap/libopenldap2_4_2/libopenldap2_4_2-2.6.3-1.tar.zst
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/cyrus-sasl/libsasl2_3/libsasl2_3-2.1.27-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/openldap/libopenldap2/libopenldap2-2.6.3-1.tar.zst
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/gsasl/libgsasl18/libgsasl18-2.2.0-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/curl/libcurl4/libcurl4-7.86.0-2.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/gnupg/gnupg-1.4.23-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/gpgme/libgpgme11/libgpgme11-1.9.0-1.tar.xz
Downloaded C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.constant.com%2f/
x86_64/release/wget/wget-1.21.3-1.tar.xz
Registry value set: HKEY_LOCAL_MACHINE\Software\Cygwin\setup\rootdir = "E:\cygwi
n64\home\benfr\cygwin64"
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/brotli/libbrotlicommon1/libbrotlicommon1-1.0.9-2.tar.
xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/c-ares/libcares2/libcares2-1.14.0-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/e2fsprogs/libcom_err2/libcom_err2-1.44.5-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libxcrypt/libcrypt2/libcrypt2-4.4.20-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/db/libdb5.3/libdb5.3-5.3.28-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libgpg-error/libgpg-error0/libgpg-error0-1.37-1.tar.x
z
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/gsasl/libgsasl-common/libgsasl-common-2.2.0-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libidn/libidn12/libidn12-1.41-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libidn2/libidn2_0/libidn2_0-2.3.4-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/krb5/libkrb5support0/libkrb5support0-1.15.2-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/nettle/libnettle6/libnettle6-3.4.1-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/nghttp2/libnghttp2_14/libnghttp2_14-1.51.0-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libntlm/libntlm0/libntlm0-1.4-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libssh2/libssh2_1/libssh2_1-1.10.0-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libunistring/libunistring2/libunistring2-0.9.10-1.tar
.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libunistring/libunistring5/libunistring5-1.1-1.tar.xz

Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libusb-win32/libusb0/libusb0-1.2.6.0-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libxml2/libxml2-2.9.12-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/zstd/libzstd1/libzstd1-1.5.2-1.tar.zst
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/noarch/release/publicsuffix-list/publicsuffix-list-dafsa/publicsuffi
x-list-dafsa-20221115-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/brotli/libbrotlidec1/libbrotlidec1-1.0.9-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libgcrypt/libgcrypt20/libgcrypt20-1.10.1-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libassuan/libassuan0/libassuan0-2.5.5-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/krb5/libk5crypto3/libk5crypto3-1.15.2-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/nettle/libhogweed4/libhogweed4-3.4.1-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libmetalink/libmetalink3/libmetalink3-0.1.3-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/libpsl/libpsl5/libpsl5-0.21.1-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/krb5/libkrb5_3/libkrb5_3-1.15.2-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/gnutls/libgnutls30/libgnutls30-3.6.9-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/krb5/libgssapi_krb5_2/libgssapi_krb5_2-1.15.2-2.tar.x
z
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/openldap/libopenldap2_4_2/libopenldap2_4_2-2.6.3-1.ta
r.zst
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/cyrus-sasl/libsasl2_3/libsasl2_3-2.1.27-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/openldap/libopenldap2/libopenldap2-2.6.3-1.tar.zst
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/gsasl/libgsasl18/libgsasl18-2.2.0-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/curl/libcurl4/libcurl4-7.86.0-2.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/gnupg/gnupg-1.4.23-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/gpgme/libgpgme11/libgpgme11-1.9.0-1.tar.xz
Extracting from file://C:\Users\benfr\Downloads/https%3a%2f%2fcygwin.mirror.cons
tant.com%2f/x86_64/release/wget/wget-1.21.3-1.tar.xz
Changing gid back to original
running: E:\cygwin64\home\benfr\cygwin64\bin\dash.exe "/etc/postinstall/0p_000_a
utorebase.dash"
running: E:\cygwin64\home\benfr\cygwin64\bin\dash.exe "/etc/postinstall/0p_updat
e-info-dir.dash"
running: E:\cygwin64\home\benfr\cygwin64\bin\bash.exe --norc --noprofile "/etc/p
ostinstall/libsasl2_3.sh"
running: E:\cygwin64\home\benfr\cygwin64\bin\bash.exe --norc --noprofile "/etc/p
ostinstall/libxml2.sh"
running: E:\cygwin64\home\benfr\cygwin64\bin\bash.exe --norc --noprofile "/etc/p
ostinstall/wget.sh"
running: E:\cygwin64\home\benfr\cygwin64\bin\dash.exe "/etc/postinstall/zp_man-d
b-update-index.dash"
Changing gid to Administrators
Ending cygwin install

```


You might see something like `Package 'wget-1.21.3-1' not found.` if the package is incorrect.
(Don't provide the version number, just the program name, in this case).

You can use this to search packages: 
```
cygcheck -p PROGRAM_NAME
```
This command only seems to work when *not* run as administrator.


