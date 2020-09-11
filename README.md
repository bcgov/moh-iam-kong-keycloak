# moh-iam-kong-keycloak
A proof of concept securing HNI services behind Kong integrated with Keycloak.

# Getting started

This is a draft Getting Started guide. I think it's complete, but we'll see. The only prerequisite is Docker.

## Run the containers

1. git clone the project
2. cd to the project directory
3. Run `docker-compose run --rm kong kong migrations bootstrap`
4. Run `docker-compose up -d`

## Add a service to Kong

```bash
curl -s -X POST http://localhost:8001/services \
    -d name=httpbin-service \
    -d url=http://httpbin.org
```

Add a route for the service:

```bash
curl -s -X POST http://localhost:8001/services/httpbin-service/routes \
    -d "paths[]=/mock"
```

Navigate to http://localhost:8000/mock and it should take you to httpbin.org.

## Configure Keycloak

First add a client:

1. Navigate to the Keycloak admin console at http://localhost:8180. Credentials are admin/admin.
2. Add Client with "Client ID" `Kong`.
3. Set the "Access Type" to `Confidential`.
4. Set "Valid Redirect URIs" to `*`.
5. Click Save.
6. Go to the "Credentials" tab. Copy the "Secret" and save it for later.

Now add a user:

1. Create a user with "Username" `user. 
2. Set "Email Verified" to `On`.
3. Set password to `password`, and "Temporary" to `Off`.

## Configure OIDC on Kong

```bash
curl -s -X POST http://localhost:8001/plugins \
  -d name=oidc \
  -d config.client_id=kong \
  -d config.client_secret=CLIENT_SECRET \
  -d config.discovery=http://HOST_IP:8180/auth/realms/master/.well-known/openid-configuration
```

Put in your host IP address and the Keycloak client secret.

Now when you navigate to http://localhost:8000/mock, you should be redirected to Keycloak for authentication.

## Tips

I ran all of the above commands in Git Bash on Windows. It has `curl` pre-installed and a handy utility for formatting JSON called `json_pp`.

```bash
curl -s http://localhost:8001 | json_pp
curl -s http://localhost:8001/services | json_pp
curl -s http://localhost:8001/routes | json_pp
curl -s http://localhost:8001/plugins | json_pp
```

If you muck up you'll probably need to delete or modify some things:

```bash
# Resource can be deleted by ID:
curl -s -X DELETE http://localhost:8001/services/edef33da-96fe-4c3d-8236-f3e35b3a0aaa
# Note that resources may also be referenced by "name" if you gave them one:
curl -s -X DELETE http://localhost:8001/services/httpbin-service
# You can modify configuration, you don't have to delete it:
curl -s -X PATCH http://localhost:8001/plugins/1e1637df-4718-4c7c-a412-4114ca29a41e --data "config.client_secret=c934568f-3fd3-4a21-bbfc-d8c7f97a3408"
```

If you need to read log files, you can use the Docker Dashboard available in the Windows task tray in the bottom right.

## References

* https://docs.konghq.com/2.1.x/admin-api
* https://www.jerney.io/secure-apis-kong-keycloak-1 (original guide, little outdated)
* https://github.com/d4rkstar/kong-konga-keycloak (newer guide, little more involved)
