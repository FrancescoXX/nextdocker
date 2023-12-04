Hi, I am Francesco Ciulla, and I used Docker for ~10 years and Next.js for 5+ years.

In this article, I want to show you how you can dockerize a Hi, I am Francesco Ciulla, and I used Docker for ~10 years and Next.js for 5+ years.

In this article, I want to show you how you can dockerize a Next.js application, and after that, I will give some considerations.

If you prefer a video version:

{% youtube HBakGXpUnjM %}

<!-- <iframe width="905" height="510" src="https://www.youtube.com/embed/HBakGXpUnjM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe> -->

All the code is available for free on [GitHub](https://youtu.be/HBakGXpUnjM) (link in video description).

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/utnf8a1n365w5qftrxhf.png)](https://youtu.be/HBakGXpUnjM)

then you can open the folder with an IDE you want.

Run `npm run dev` to run your Next.js app

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/eb32m7kn5sox2k5x0vei.png)](https://youtu.be/HBakGXpUnjM)

And if you visit `localhost:3000` you will see the Next.js app running.

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otjb9jisovafdh01ol8g.png)](https://youtu.be/HBakGXpUnjM)

# Dockerize the Next.js app

Now we can dockerize the Next.js app.

Open the file called `next.config.js` and replace the content with this:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone'
}

module.exports = nextConfig
```

## .dockerignore

Create a file called `.dockerignore` and add this content:

```
Dockerfile
.dockerignore
node_modules
npm-debug.log
README.md
.next
.git
```

This is needed to avoid copying the `node_modules` folder to the Docker image.

In case you want an explanation, go [here](https://youtu.be/HBakGXpUnjM?si=WdSpoiqqV51zo3KI&t=246)

## Dockerfile

At the root of the project, create a file called `Dockerfile` and add this content:

```dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then yarn global add pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi


# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Next.js collects completely anonymous telemetry data about general usage.
# Learn more here: https://nextjs.org/telemetry
# Uncomment the following line in case you want to disable telemetry during the build.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN yarn build

# If using npm comment out above and use below instead
# RUN npm run build

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
# Uncomment the following line in case you want to disable telemetry during runtime.
# ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
# set hostname to localhost
ENV HOSTNAME "0.0.0.0"

# server.js is created by next build from the standalone output
# https://nextjs.org/docs/pages/api-reference/next-config-js/output
CMD ["node", "server.js"]
```

In case you want an explanation, go [here](https://youtu.be/HBakGXpUnjM?si=GrbrMX-2t1f0oWDb&t=358)

## Docker compose

Create a file called `docker-compose.yml` and add this content:

```yml
version: '3.9'

services:
  nextapp:
    container_name: nextapp
    image: nextapp
    build: .
    ports:
      - "3000:3000"
```

In case you want an explanation, go [here](https://youtu.be/HBakGXpUnjM?si=Ek_5jbwW4GmVKOpg&t=474)

## Build the Docker Image and run the service

Build the Docker image:

```bash
docker compose build
```

Then run the services

```
docker compose up 
```

And you should see something like this

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otqbjbxd10kv85aw1wws.png)](https://youtu.be/HBakGXpUnjM)

If you visit `localhost:3000` you will see the Next.js app running (but this time using Docker)

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otjb9jisovafdh01ol8g.png)](https://youtu.be/HBakGXpUnjM)


## Livecycle

As final test, I want to try Livecycle to deply the app

You can just download the extension on Docker Desktop

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ldklewmwrvvo4x0yx3qi.png)](https://youtu.be/HBakGXpUnjM)

When your app is running locally, you cna just use Docker Desktop to turn it on.

[![Next js and Docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4anje4ck6m95g6srr6g4.png)](https://youtu.be/HBakGXpUnjM)

It's super effective!


# Considerations:
- Docker is a technology that can work on ANY application, of course including Next.js.
- Next.js has a super cool way to deploy applications, to make Docker ALMOST useless.
- Having Next.js running in a Docker container they provide a better security and dependability, faster and easier deployment procedures, and simpler management of the application.
- If you already have many Docker containers running, it's easier to add a new one instead of using the Next.js deployment procedure (Vercel), to hace the full devops control.
- If you have ONLY one Next.js application, I would suggest using the Next.js deployment procedure (Vercel).

So do you need to use Docker with Next.js? Usually NOT, but if you have multiple services running (and you use a tool like Livecycle), it starts to make sense

This makes us think how Docker is powerful. Docker doesn't care about the technology you use, it just works. With Docker you cna deploy ANY application, and this is super powerful.

Kudos to Vercel for making the Next.js app deployment so easy, but I think that Docker is still a great tool to have.

If you prefer a video version:

{% youtube HBakGXpUnjM %}

<!-- <iframe width="905" height="510" src="https://www.youtube.com/embed/HBakGXpUnjM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe> -->

All the code is available for free on [GitHub](https://youtu.be/HBakGXpUnjM) (link in video description).




