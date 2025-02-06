# AI4Edu Deployment manual

This manual covers the steps taken to deploy the AI4Edu application on a self-owned domain.

Updated on Febuary 5th 2025.

## 1. Getting a SSL certificate

To host the server on a HTTPS connection a SSL certificate is needed.

To acquire one go to https://certbot.eff.org/instructions?ws=nginx&os=pip (this link is specific to a server running on nginx and linux) and follow the installation instructions.

Once completed the certificate keys should be saved in /etc/letsencrypt/live/*{domain}*

## 2. File configurations

For the file configurations the ports have to be consistent. The project has changed its frontend port from 3100 to 3110 on recent branches but here it is assumed that the frontend port is 3100.

- ### .env file

The following environment variables should be specified:

`SERVER_HOST='{domain}'`

`REVERSE_PROXY_FRONTEND=3100`

`ALLOWED_ORIGINS='{domain} {domain}:3100 {domain}:8686 {domain}:8687'`

As well as the following variables for secret keys (fill in your own):

`OPENAI_API_KEY=...`

`MINIO_ACCESS_KEY=...`

`MINIO_SECRET_KEY=...`

- ### /chatbot-ui/frontend/src/global.jsx file

The exported URL variables should be altered to include your domain name:

`export const BACKEND_URL = 'https://{domain}:8687';`

`export const SOCKET_URL = '{domain}:8687';`

- ### docker-compose.yaml file

In the reverse-proxy service make sure that the ports match:

 `- "${REVERSE_PROXY_FRONTEND:-3100}:3100"`

And that its volumes point to the correct path for the certificate keys:

`- /etc/letsencrypt/live/{domain}/fullchain.pem:/app/fullchain.pem`

`- /etc/letsencrypt/live/{domain}/privkey.pem:/app/privkey.pem`

- ### nginx_config.conf file

Finally make sure that the http server listens to the correct frontend port:

`listen 3100 ssl;`

## 3. Deploying

If logged in to the server host via ssh, you should keep the server running after disconnecting by running it in a screen session.

To create a new screen session, run:

`screen -S {session name}`

Then deploy the project by running:

`docker compose up --build`

The platform should then be accessible on `https://{domain}:3100/ai4edu/`.

If server issues occur, check if the screen session is still active by running:

`screen -ls`

If it is, you can inspect the server issues by resuming the screen session:

`screen -r {session name}`
