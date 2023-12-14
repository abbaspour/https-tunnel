So this is the setup. We want to know what's an application that only accepts secure connection doing:


![Stunnel Diagram](https://gist.githubusercontent.com/abbaspour/ac027a167445aa8d3c08bde4d46d4a6b/raw/7094eecbf402c6a60c076e4cc46db7932df950e1/_stunnel.png)

```bash
brew install socat stunnel
```

## Self Signed
Create a self-signed certificate and install in into the system:

```bash
openssl req -new -x509 -days 365 -nodes -out cert.pem -keyout cert.pem
chmod 600 cert.pem
```

## Let's Encrypt
Follow certbot [instructions](https://certbot.eff.org/instructions).
```bash
brew install certbot

# First time
certbot certonly --standalone \
  --http-01-port=3000 \
  --logs-dir . --config-dir . --work-dir . \
  --agree-tos --email my@email.com \
  --domains mydomain.com 
  
# Renewal
certbot renew \
  --logs-dir . --config-dir . --work-dir .

cat ./live/mydomain.com/privkey.pem ./live/mydomain.com/fullchain.pem > cert.pem
chmod 600 cert.pem
 
```
then run it:

```bash
stunnel listener.conf
```

> This is where you can STOP and enjoy HTTPS to your application. Proceed only if you want to tail traffic.

```bash
stunnel client.conf
```

So in between listener and client stunnel instances, we run socat to monitor the traffic:

```bash
socat -v tcp-listen:1080,reuseaddr,fork,keepalive tcp:localhost:1081
```

That's it folks. try accessing localhost:1443 over HTTPS and you can see the plain traffic in the socat terminal.

```bash
wget -O - --no-check-certificate https://localhost:1443/
```

### Notes ###

Q1: but I get '''HTTP 404''' all the time? try adding hostname to /etc/hosts. Server name in HTTP header should match

```bash
echo "127.0.0.1 www.twitter.com" >> /etc/hosts
echo "127.0.0.1 www.google.com.au" >> /etc/hosts
```

