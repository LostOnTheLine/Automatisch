# Automatisch
A sample compose for Automatisch using the pre-built image instead of building it every time the container starts<br>
Since we're using the image instead of building it at start like the github uses you have to disable the `ENTRYPOINT:` variable. I've left in in just commented out in case there's a problem & you want to know the original. <br>
You also have to set a variable for `ENCRYPTION_KEY`, `WEBHOOK_SECRET_KEY`, & `APP_SECRET_KEY` or it won't work <br>
- I've changed the exposed port & changed the containers names because I hate things that are just "main" & "worker"
- I've added the `container_name` variable as I always prefer to have those declared
- I've made everything a Bind Mount instead of Volumes because I think Volumes are dumb for most people's usecase. <h6>Other's disagree on that but if you prefer volumes use the lines from the original</h6>
- I've added my Watchtower labels for updates. You will need to adjust those to your own setup, but if you don't use it it won't cause a problem
- I've also unified the order the way I like it to maintain consistency with my other stacks
  - ###### image - container_name - depends_on - environment - ports - volumes - restart - healthcheck - labels <br>
**All those things are obviously not required.** 

### For initial Login use 
- **Email**: user@automatisch.io
- **Password**: sample


```yaml
version: '3.9'
services:
  automatisch-main:
    image: ghcr.io/automatisch/automatisch:latest
    container_name: automatisch-main
    #build:
    #  context: ./docker
    #  dockerfile: Dockerfile.compose
    #entrypoint: /compose-entrypoint.sh
    depends_on:
      automatisch-postgres:
        condition: service_healthy
      automatisch-redis:
        condition: service_started
    environment:
      - HOST=localhost
      - PROTOCOL=http
      - PORT=8123
      - APP_ENV=production
      - REDIS_HOST=automatisch-redis
      - POSTGRES_HOST=automatisch-postgres
      - POSTGRES_DATABASE=automatisch
      - POSTGRES_USERNAME=<<YourPostgresUser>>
      - POSTGRES_PASSWORD=<<YourPostgresPassword>>
      - ENCRYPTION_KEY=<<YourENCRYPTIONkey>>
      - WEBHOOK_SECRET_KEY=<<YourWEBHOOKsecretKey>>
      - APP_SECRET_KEY=<<YourAPPsecretKey>>
    ports:
      - '8123:8123'
    volumes:
      - /docker/zapier/automatisch/automatisch_storage:/automatisch/storage
    restart: unless-stopped
    labels:
      - "com.centurylinklabs.watchtower.scope=github"
 ##################################################
  automatisch-worker:
    image: ghcr.io/automatisch/automatisch:latest
    container_name: automatisch-worker
    #build:
    #  context: ./docker
    #  dockerfile: Dockerfile.compose
    #entrypoint: /compose-entrypoint.sh
    depends_on:
      - automatisch-main
    environment:
      - APP_ENV=production
      - REDIS_HOST=automatisch-redis
      - POSTGRES_HOST=automatisch-postgres
      - POSTGRES_DATABASE=automatisch
      - POSTGRES_USERNAME=<<YourPostgresUser>>
      - POSTGRES_PASSWORD=<<YourPostgresPassword>>
      - ENCRYPTION_KEY=<<YourENCRYPTIONkey>>
      - WEBHOOK_SECRET_KEY=<<YourWEBHOOKsecretKey>>
      - APP_SECRET_KEY=<<YourAPPsecretKey>>
      - WORKER=true
    volumes:
      - /docker/zapier/automatisch/automatisch_storage:/automatisch/storage
    restart: unless-stopped
    labels:
      - "com.centurylinklabs.watchtower.scope=github"
 ##################################################
  automatisch-postgres:
    image: 'postgres:14.5'
    container_name: automatisch-postgres
    environment:
      - POSTGRES_DB=automatisch
      - POSTGRES_USER=<<YourPostgresUser>>
      - POSTGRES_PASSWORD=<<YourPostgresPassword>>
    volumes:
      - /docker/zapier/automatisch/postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}']
      interval: 10s
      timeout: 5s
      retries: 5
    labels:
      - "com.centurylinklabs.watchtower.scope=dockerhub"
 ##################################################
  automatisch-redis:
    image: 'redis:7.0.4'
    container_name: automatisch-redis
    volumes:
      - /docker/zapier/automatisch/redis_data:/data
    restart: unless-stopped
    labels:
      - "com.centurylinklabs.watchtower.scope=dockerhub"

```
