# bookworm was only released on 10-6-2023, so is a little too new.
FROM node:18-bullseye AS builder
# global installs need root permissions, so have to happen before we switch to
# the node user
RUN npm i -g pnpm@8
# node images create a non-root user that we can use
USER node
WORKDIR /home/node/build

COPY --chown=node:node . .

ARG HOME_LOCATION
ARG API_LOCATION
ARG FORUM_LOCATION
ARG NEWS_LOCATION
ARG RADIO_LOCATION
ARG CLIENT_LOCALE
ARG CURRICULUM_LOCALE
ARG SHOW_LOCALE_DROPDOWN_MENU
ARG ALGOLIA_APP_ID
ARG ALGOLIA_API_KEY
ARG STRIPE_PUBLIC_KEY
ARG PAYPAL_CLIENT_ID
ARG PATREON_CLIENT_ID
ARG DEPLOYMENT_ENV
ARG SHOW_UPCOMING_CHANGES
ARG SHOW_NEW_CURRICULUM
ARG GROWTHBOOK_URI
ARG FREECODECAMP_NODE_ENV

# We're installing specific packages even though it is not strictly necessary -
# pnpm install would work. The idea is to make the dependencies explicit and
# keep them under our control.
RUN pnpm config set dedupe-peer-dependents false
# Scripts need to be run at this stage (--ignore-scripts cannot be used) because
# without them, Gatsby will not install sharp.
RUN pnpm install -w -F=client -F=ui -F=browser-scripts -F=challenge-parser \
  --frozen-lockfile
RUN pnpm build:client

FROM node:18-alpine
RUN npm i -g serve@13 pm2@4
USER node
WORKDIR /home/node/client
COPY --from=builder /home/node/build/client/public/ public
COPY --from=builder /home/node/build/docker/web/serve.sh serve.sh
COPY --from=builder /home/node/build/docker/web/pm2-start.sh pm2-start.sh
COPY --from=builder /home/node/build/docker/web/serve.json serve.json

ENTRYPOINT [ "./pm2-start.sh" ]
CMD [ "8000" ]
