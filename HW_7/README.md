# Simple PKI and Tomcat TLS Deployment

This project implements a complete Public Key Infrastructure (PKI) using OpenSSL, based on the structure from the [Simple PKI tutorial](https://pki-tutorial.readthedocs.io/en/latest/simple/), and deploys a TLS certificate on an Apache Tomcat 10 server for the domain `www.simple.org`.

---

## ✅ Project Features

- Offline Root CA (self-signed)
- Online Signing (Intermediate) CA
- Server certificate for `www.simple.org`
- CRL (Certificate Revocation List) support
- TLS deployed on Apache Tomcat 10 using JSSE-compatible PKCS#12 keystore
- Verified with curl and openssl

---

## 🔧 Prerequisites

- OpenSSL 1.1.1 or higher (macOS/Linux)
- Java 17+ (e.g. OpenJDK 17 or later)
- Apache Tomcat 10.1.24
- Bash shell (for scripts)
- curl and keytool

---

## 🚀 Setup Instructions

### 1. Clone the Repo

```bash
git clone https://github.com/your-username/pki-example-1.git
cd pki-example-1
```

### 2. Create the Root and Signing CA

```bash
# From project root
./scripts/00_init_dirs.sh
./scripts/10_create_root.sh
./scripts/20_create_signing_ca.sh
```

### 3. Issue Server Certificate

```bash
./scripts/30_issue_server_cert.sh
```

### 4. Generate the CRL

```bash
openssl ca -gencrl -config etc/signing-ca.conf -out crl/signing-ca.crl
```

### 5. Export Server Cert to Keystore

```bash
openssl pkcs12 -export   -inkey certs/www.simple.org.key   -in certs/www.simple.org.crt   -certfile ca/signing-ca.crt   -name tomcat   -out certs/tomcat.jks   -passout pass:mypassword
```

---

## 🔐 Tomcat HTTPS Setup

### 1. Download and Extract Tomcat

```bash
wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.24/bin/apache-tomcat-10.1.24.tar.gz
tar -xzf apache-tomcat-10.1.24.tar.gz
```

### 2. Replace server.xml

```bash
cp tomcat-conf/server.xml apache-tomcat-10.1.24/conf/server.xml
```

### 3. Start Tomcat

```bash
cd apache-tomcat-10.1.24
./bin/startup.sh
```

---

## 🔎 Verification

### Test with curl

```bash
curl -vk https://localhost:8443 --cacert ca/root-ca.crt
```

### Test with OpenSSL

```bash
openssl s_client -connect localhost:8443 -servername www.simple.org -showcerts
```

---

## 📝 Screenshots Required

- Root CA cert creation
- Signing CA cert issued by Root
- Server cert issued by Signing CA
- Keytool listing showing PrivateKeyEntry
- Tomcat logs: “Starting ProtocolHandler [https-jsse-nio-8443]”
- curl success with HTTPS and valid cert

---

## 🛡️ Security Practices

- Root key kept offline
- CRLs published and verifiable
- PKCS12 keystore uses same password for store and key
- Tomcat SSLHostConfig used (no outdated attributes)

---

