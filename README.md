# No-IP Linux DUC for Docker
<div align="center">

![Version](https://img.shields.io/badge/dynamic/yaml?url=https://raw.githubusercontent.com/tgcentral/linux-update-client-docker/main/.github/workflows/deploy.yml&query=$.env.VERSION&label=version&style=for-the-badge&color=#8fbe00)
![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/tgcentral/linux-update-client-docker/deploy.yml?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-docker-blue?style=for-the-badge)
![GitHub License](https://img.shields.io/github/license/tgcentral/linux-update-client-docker?style=for-the-badge&color=#8fbe00)

**🚀 Automated Dynamic DNS Updates Made Simple**

*Keep your domain name pointing to your current IP address, automatically*

</div>

---

> 🔒 **About this fork**
>
> This is a fork of [noipcom/linux-update-client-docker](https://github.com/noipcom/linux-update-client-docker). The upstream image `ghcr.io/noipcom/noip-duc` is built on a Debian base that hasn't been rebuilt since its last release, so vulnerability scanners flag Critical/High CVEs in its OS packages (OpenSSL, GnuTLS, PCRE2, PAM, Perl and others). The findings are all in Debian packages, not in the No-IP client code.
>
> This fork builds the same No-IP client, but installs the latest Debian security updates into the image at build time. It rebuilds weekly so new fixes are picked up automatically, and publishes to `ghcr.io/tgcentral/noip-duc`. Each build is tagged `:latest` and `:<version>-<YYYYMMDD>`. See [docs/plan-option-a.md](docs/plan-option-a.md) for the background.

## 📋 Description
The No-IP update client for Docker is a handy tool that automates the process of keeping your dynamic IP address in sync with your No-IP hostname(s). In simpler terms, it helps you make sure that your domain name (like `camera.ddns.net`) always points to your current IP address.

> 💡 This docker image is based on our [Official Linux DUC version 3](https://www.noip.com/download?page=linux), providing enterprise-grade reliability in a containerized format.

## 🚀 Quick Start

### 1. Pull the docker container from Github Container Registry (GHCR)

``` bash
docker pull ghcr.io/tgcentral/noip-duc:latest
```

### 2. Create Your Configuration

Create an `.env` file (e.g. `noip-duc.env`) in a secure location with your No-IP credentials: 

> ⚠️ **Security Note**: Set appropriate permissions on your env file, ideally `0600`

``` bash
# noip-duc.env with DDNS Key
NOIP_USERNAME=DdnsKeyUser
NOIP_PASSWORD=DdnsKeyPass
NOIP_HOSTNAMES=all.ddnskey.com
```

**Reminder**: Replace `DdnsKeyUser`, `DdnsKeyPass`, and the hostnames with your actual No-IP account credentials and hostname or `all.ddnskey.com` as the hostname if you are using a DDNS Key.

### 3. Run the Container

```bash
docker run -d --env-file noip-duc.env --name noip-duc ghcr.io/tgcentral/noip-duc:latest
```

> 💡 **Pro Tip**: Run `docker run noip-duc --help` to see all available environment variables and options.



### Docker Compose

Thanks for the help from our contributors, we now include an example `compose.yaml` file in the repository.

Simply run `docker compose up` once you have filled in the noip-duc.env file with your credentials.


---

## 🔐 Authentication Methods

### DDNS Keys (Recommended)

[DDNS Keys](https://www.noip.com/support/knowledgebase/how-to-setup-and-use-a-ddns-key) provide enhanced security with randomly generated credentials for each hostname.

Once you have created a DDNS Key on your No-IP account, put it in your configuration (`.env` file). 

If your DDNS Key Username is `yf5f8n5` and your DDNS Key Password is `gHil56Bu`, your configuration file should look like this: 

``` bash
# noip-duc.env with DDNS Key
NOIP_USERNAME=yf5f7n5
NOIP_PASSWORD=gHil56Bu
NOIP_HOSTNAMES=all.ddnskey.com
```

> ✨ **Note**: With DDNS Keys, use `all.ddnskey.com` as the hostname - no need to specify individual hostnames!

### 👥 Groups (Legacy)

For existing Groups configurations:
```bash
# noip-duc.env with Groups
NOIP_USERNAME=mygroup:myuser
NOIP_PASSWORD=GroupPassword
NOIP_HOSTNAMES=myhostname.ddns.net
```

---

## 🌐 IPv6 Support

Enable IPv6 updates by creating an AAAA (IPv6) type hostname and adding this to your ENV file:

```bash
NOIP_IP_METHOD=http://ip1.dynupdate6.no-ip.com/
```

> 📖 Learn more about [IPv6 hostname creation](https://www.noip.com/support/knowledgebase/creating-ipv6-aaaa-host/)

---

## 🛠️ Management

### Check Container Status
```bash
docker logs noip-duc
```

### Update Credentials
1. Edit your environment file with new values
2. Restart the container:
```bash
docker restart noip-duc
```

### Get Help
```bash
docker run noip-duc --help
```

---

## 🤝 Support & Contribution

![GitHub Issues](https://img.shields.io/github/issues/noipcom/linux-update-client-docker?style=for-the-badge)
[![Support Ticket](https://img.shields.io/badge/Support-Create%20Ticket-blue?style=for-the-badge)](https://www.noip.com/ticket)

### Need Help?
- Check out our [Knowledge Base](https://www.noip.com/support/knowledgebase) for extensive documentation on all things No-IP
- Create a [Support Ticket](https://www.noip.com/ticket) for technical assistance
- Contact us at [support@noip.com](mailto:support@noip.com)

### Want to Contribute?
We welcome contributions! Feel free to:
- [Submit an issue](https://github.com/noipcom/noip-duc/issues/new) for bugs
- [Create a pull request](https://github.com/noipcom/noip-duc/pulls) for improvements

<div align="center">

**Made with ❤️ by the No-IP Team**
</div>
