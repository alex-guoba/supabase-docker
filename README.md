# Supabase Docker

A simple Docker setup for Supabase.

## Features

- Remove unnecessary services (`realtime`, `storage`, `function`, `imgproxy`) to improve performance.
- Add support for OAuth(`github`, `twitter`) providers.
- Add support for EMail Template.
- Add SSL support.

## Getting started

### Prerequisites

- Docker
- Docker Compose
- Git
- Node

### Installation

```bash
git clone https://github.com/alex-guoba/supabase-docker.git
cd supabase-docker
cp .env.example .env

# To generate secrets and API keys into `.env`:
sh ./utils/generate-keys.sh
sh ./utils/add-new-auth-keys.sh

# Update .env file to your needs

#SUPABASE_PUBLIC_URL=http://<your-supabase-domain>
#API_EXTERNAL_URL=http://<your-supabase-domain>
#SITE_URL=https://<your-application-domain>

# Start Supabase
docker compose up -d

# test
sh ./tests/test-self-hosted.sh

# Stop Supabase
docker compose down
```

### Usage

After running `docker compose up -d`, you can access Supabase at http://127.0.0.1:8000.

## Configuration

You can configure Supabase by editing the `.env` file.

### Basic Configuration

- `SITE_URL`: The URL of your Application.
- `API_EXTERNAL_URL`: The URL of your Supabase Instance.

### Auth Provider

Add OAuth provider to auth section of `docker-compose` file. For example, to add Github OAuth provider:

```yaml
# Github OAuth config in auth.enviroment
# Comment to disable this
GOTRUE_EXTERNAL_GITHUB_ENABLED: ${GITHUB_ENABLED}
GOTRUE_EXTERNAL_GITHUB_CLIENT_ID: ${GITHUB_CLIENT_ID}
GOTRUE_EXTERNAL_GITHUB_SECRET: ${GITHUB_SECRET}
GOTRUE_EXTERNAL_GITHUB_REDIRECT_URI: ${API_EXTERNAL_URL}/auth/v1/callback
```

Note that you need to set `GITHUB_CLIENT_ID` and `GITHUB_SECRET` in your `.env` file:

```
GITHUB_ENABLED=true
GITHUB_CLIENT_ID=
GITHUB_SECRET=
```

After adding the provider, You can use the API to verify the configuration of your OAuth providers:

```bash
curl 'https://<PROJECT_REF>.supabase.co/auth/v1/settings' \
-H "apikey: <ANON_KEY>" \
-H "Authorization: Bearer <ANON_KEY>"
```

or you can input the `Auth Providers` in Commmand Menu of Supabase to check the configuration.

### SMTP

Use SMTP to send emails. In your `.env` file, add the following to configure your SMTP server:

```shell
ENABLE_EMAIL_SIGNUP=true
ENABLE_EMAIL_AUTOCONFIRM=false
SMTP_ADMIN_EMAIL=admin@example.com
SMTP_HOST=supabase-mail
SMTP_PORT=2500
SMTP_USER=fake_mail_user
SMTP_PASS=fake_mail_password
SMTP_SENDER_NAME=fake_sender
ENABLE_ANONYMOUS_USERS=false
```

See [Supabase SMTP](https://supabase.com/docs/guides/auth/auth-smtp) for more details.

### Email Templates

You can customize the email templates by editing the files in `./docker-compose.yml`.

```yml
GOTRUE_MAILER_TEMPLATES_RECOVERY: ${MAILER_TEMPLATES_RECOVERY}
GOTRUE_MAILER_TEMPLATES_INVITE: ${MAILER_TEMPLATES_INVITE}
GOTRUE_MAILER_TEMPLATES_CONFIRMATION: ${MAILER_TEMPLATES_CONFIRMATION}
GOTRUE_MAILER_TEMPLATES_MAGIC_LINK: ${MAILER_TEMPLATES_MAGIC_LINK}
GOTRUE_MAILER_TEMPLATES_EMAIL_CHANGE: ${MAILER_TEMPLATES_EMAIL_CHANGE}
```

Set your environment value in `.env`:

```shell
MAILER_TEMPLATES_INVITE="https://example.com/templates/invite.html"
MAILER_TEMPLATES_CONFIRMATION="https://example.com/templates/confirm.html"
MAILER_TEMPLATES_RECOVERY="https://example.com/templates/recovery.html"
MAILER_TEMPLATES_MAGIC_LINK="https://example.com/templates/magic-link.html"
MAILER_TEMPLATES_EMAIL_CHANGE="https://example.com/templates/email-change.html"
```

The email templates are written in HTML and can be customized to fit your needs. See [Supabase Email Templates](https://supabase.com/docs/guides/auth/auth-email-templates) for more details.

### SSL Configuration

#### Method 1: Use your own certificate with kong in supabase instance

To enable SSL, you need to generate your own certificate and key files for Kong instance in supabase.
Generate your cert and key files and copy it to `./volumns/ssl/` directory. In your `.env` file, add the following:

```shell
SSL_CERT="example.crt"
KONG_SSL_CERT_KEY="example.key"
```

Uncomment the following line in `docker-compose.yml`:

```yaml
KONG_SSL_CERT: /mnt/ssl/${SSL_CERT_FILENAME}
KONG_SSL_CERT_KEY: /mnt/ssl/${SSL_CERT_KEY_FILENAME}
```

#### Method 2: Use reverse proxy and let's encrypt

Adding a reverse proxy with automatic HTTPS in front of your self-hosted Supabase, you can use nginx as a reverse proxy.

Change your `.env` file for nginx:

``shell
PROXY_DOMAIN=your-domain.example.com
CERTBOT_EMAIL=admin@your-domain.example.com

````

Start Nginx by using the pre-configured docker-compose.nginx.yml override:

```shell
docker compose -f docker-compose.yml -f docker-compose.nginx.yml up -d
````

More details can be found in [Supabase Reverse Proxy with HTTPS](https://supabase.com/docs/guides/self-hosting/self-hosted-proxy-https).

### Other Configuration

API Keys, Secrets, and other configurations can be found in the official Supabase documentation. Make sure to update your `.env` file accordingly.

See [Docker Configuration](https://supabase.com/docs/guides/self-hosting/docker) for more details.

## Development

All development is done in the `main` branch.
There is a `develop` docker file for development. You can override the `docker-compose.yml` file to use it:

```
docker compose -f docker-compose.yml -f ./dev/docker-compose.dev.yml up
```

## Deploy in Dokploy

[Dokploy](https://dokploy.com/) is a stable deployment platform for Docker applications. It is a simple way to deploy your Docker application to the cloud.

Dokploy provide some [templates](https://docs.dokploy.com/docs/core/templates) for all sort of Docker applications. we can use it to deploy Supabase.

If you want some customization, you can deploy through your `docker-compose.yml` file. Here is an [example](./docker-compose.dokploy.yml).

Here is the process to deploy Supabase from `docker-compose.yml` file.

1. Create a new project in Dokploy.
2. Create a Compose service in project.
3. Configure the deployment.

- Provider: set your github repo in github provider. or you can paste the contents of `./docker-compose.dokploy.yml` in raw provider.
- Environment: paste the contents of `./.env` in environment.
- Volumes: add all of volume files in volumes setting. for example:
  - Content: content of [vector.sql](./volumes/logs/vector.yml)
  - Path: set to `/volumes/logs/vector.yml`
- Docker links: enable `Isolate Deployment` in Utilities configuration.
- Domains: set your domain in domains.

4. Deploy.

## License

[MIT](https://choosealicense.com/licenses/mit/)

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
