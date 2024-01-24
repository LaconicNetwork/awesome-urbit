# Self Hosted DeFi Front Ends

TL;DR: `|install ~lanfyn-dasnys %uniswap`

Applications in web3 still remain significantly centralized. Usually, the blockchain used by a particular application cannot easily be censored. However, front ends are always served to users via a centralized pipeline of service providers, each of which is a choke point for that application.

On Urbit, an application you install on your ship is always available to you. It will be available to others on the network if you publish it to your ship. However, it will only remain available to others if your ship is online. What was once an easy to install application can easily become unavailable. Indeed, this is what happened to the Osmosis and Uniswap front ends on Urbit.

Laconic provides a various solutions to existing web3 applications that require increased robustness and jurisdictional diversity (read: decentralization), thus the natural alignment with Urbit. The following guide uses Laconic's 'Stack Orchestrator' tool to demonstrate the ease with which any web3 application front end can be easily integrated into existing CI/CD workflows, in order to publish and maintain a front end on Urbit.

Broadly, the steps are as such:

- Modify app to conform with Urbit requirements
- Generate and host a glob file
- Publish app to your ship

The first step is very much application dependent but only needs to be done once. Steps 2 and 3 are easily automated via familiar CI/CD pipelines. This tutorial outlines how to add any DeFi application to Urbit and gives examples of doing so for both Uniswap and Osmosis. Background reading of these Urbit docs is helpful as background.

- https://docs.urbit.org/manual/getting-started/get-id
- https://docs.urbit.org/manual/getting-started/self-hosted/cli
- https://docs.urbit.org/userspace/apps/guides/software-distribution
- https://docs.urbit.org/userspace/apps/reference/dist (esp. the `glob` section)
- https://docs.urbit.org/courses/app-school-full-stack/8-desk

## Modify App

Your app front end must comply with a variety of Urbit requirements. The first step is to ensure that you can compile a static build of your application. This build will be consumed by the Urbit `globulator` and these files will need to be located in the root directory of your Urbit ship (usually a planet). More on this later. You might already have `yarn build:static` or an existing way to generate a static build for your app. If not, ensure your application can run as intended when served as a static build.

The next step is to ensure a few things about the files, paths, and URL in your app. First, all the uppercase letters in all the file names of your app need to be converted to lowercase. As well, ensure that the file paths and URL does not contain square brackets. Next.js can do awkward things in this regard, for example, with the dynamic path rendering.

Next, you’ll need to generate Urbit mark files that are missing from the default `%landscape` desk. For simple applications, this should not be necessary. Every file extension in your application requires a corresponding mark file. These are short files written in Hoon that are required for the globulator to function correctly. By inspecting existing mark files, you should get a good idea as to what a new one needs to look like if your application contains any exotic file extensions.

Finally, you’ll have to decide what to do with external API calls and other services that your app uses. In the case of Uniswap, we run a proxy server and forward requests to the original application. Other solutions to address this issue are outside the scope of this blog post.

You can view the modifications required to the Uniswap front end in [our fork here](https://github.com/cerc-io/uniswap-interface/pulls?q=is%3Apr+is%3Aclosed). These changes aren’t upstreamed, therefore publishing new versions to Urbit would require manually rebasing and addressing any merge conflicts, followed by re-globbing and updating the `desk.docket-0` file, as described next. For your application, this process can easily be automated using existing CI/CD workflows.

## Globulate

The front end of an Urbit app is packaged up into something called a `glob`. This glob can either be served over http or ames. The Urbit documentation has great examples using ames and the globulator UI. For ease of automation, we use http and glob from a bash script that sends `curl` requests to a running fakezod.

The static build of the application are the files that need to be globbed. A directory of these files needs to be located in the root of your ships directory. Under the hood, we’ve abstracted away the majority of this part.

### Glob Hosting

An http glob can be hosted wherever you want, like Amazon S3 or Digital Ocean Spaces Object Storage. The Laconic solution includes by default a locally running ipfs node to which the glob is published.

### desk.docket-0

The tile for each app that you see when logging into your Urbit is defined by the desk.docket-0. Therefore, adding CI/CD workflows to a traditional app for Urbit requires updating this file and re-publishing an application.

## Install and Publish

This part is ensuring that your desk.docket-0 is correct then running `|install our %uniswap` in the Urbit dojo. The app should now be available as a tile in Landscape. To make it available for others to install from your ship, run `:treaty|publish %uniswap` in the dojo.

To separate the development and production workflows, we use a fakezod for reviewing and testing modifications, then when ready to publish to the network, use a deployment script directed at a live planet. That's how Uniswap was made available to anyone on the Urbit network. Run `|install ~lanfyn-dasnys %uniswap` in your dojo.

## Demo

### Install Stack Orchestrator

```
git clone https://github.com/cerc-io/stack-orchestrator.git
cd stack-orchestrator
./scripts/quick-install-linux.sh
```

press Y and follow the instructions at the end, then;

```
laconic-so version
```

should look like:

```
Version: 1.1.0-cef73d8-202401231732
```

With a handful of new concepts involved in Urbit app development, automated DeFi deployments happened to be a great fit for the Laconic Stack Orchestrator tool. We’ve distilled the above steps into a few commands that can be run by anyone on a stock Digital Ocean. The following instructions will build and deploy the Uniswap front end to a fakezod. `laconic-so` has specific "stacks" that are defined by a `stack.yml`. For the Uniswap Urbit App, it looks like this:

```yaml
version: "0.1"
name: uniswap-urbit-app
repos:
  - github.com/cerc-io/uniswap-interface@laconic
  - github.com/cerc-io/watcher-ts@v0.2.78
containers:
  - cerc/uniswap-interface
  - cerc/watcher-ts
pods:
  - uniswap-interface
  - proxy-server
  - fixturenet-urbit
  - kubo
```

We'll be building two docker images; one for the app and one for the proxy server. Urbit and Kubo (ipfs) are run using the default docker images.

### Setup

First, clone the required repositories:

```
laconic-so --stack uniswap-urbit-app setup-repositories
```

The output looks like:

```
Dev Root is: /root/cerc
Dev root directory doesn't exist, creating
Checking: /root/cerc/uniswap-interface: Needs to be fetched
100%|#####################################################################################################################| 43.8k/43.8k [00:05<00:00, 7.77kB/s]
switching to branch laconic in repo cerc-io/uniswap-interface
Checking: /root/cerc/watcher-ts: Needs to be fetched
100%|#####################################################################################################################| 12.2k/12.2k [00:01<00:00, 6.82kB/s]
switching to branch v0.2.78 in repo cerc-io/watcher-ts
```

You can see we cloned the two repos and switched to the branch/tag/version specified in the `stack.yml`

### Build

```
laconic-so --stack uniswap-urbit-app build-containers
```

This can take awhile and will produce a ton of output; if successful, you'll see something like:

```
Successfully built fa34caca25f1
Successfully tagged cerc/watcher-ts:local
```

at the end.

The `uniswap-interface` image is simple; it installs the dependencies for our modified version of Uniswap. The static build will be produced at deploy time.

```
FROM node:18.17.1-alpine3.18

RUN apk --update --no-cache add git make alpine-sdk bash

WORKDIR /app

COPY . .

RUN echo "Building uniswap-interface" && \
    yarn
```

The `watcher-ts` image is used for the proxy server. By default, this proxy server re-directs requests back to Uniswap. This means that the Uniswap front end on Urbit requires api.uniswap.org to be up and running. The configuration also takes an optional Infura API Key, which would be required for power users or an increase in traffic using the application.

### Create Deployment

First, create a spec file for the deployment:

```
laconic-so --stack uniswap-urbit-app deploy init --output uniswap-urbit-app-spec.yml
```

edit `uniswap-urbit-app-spec.yml` so that it looks exactlylike:

```yaml
stack: uniswap-urbit-app
deploy-to: compose
network:
  ports:
    proxy-server:
      - '4000:4000'
    urbit-fake-ship:
      - '8080:80'
    ipfs:
     - '4001'
     - '8081:8080'
     - 0.0.0.0:5001:5001
volumes:
  urbit_app_builds: ./data/urbit_app_builds
  urbit_data: ./data/urbit_data
  ipfs-import: ./data/ipfs-import
  ipfs-data: ./data/ipfs-data
```

Save your changes then create a deployment from that file:

```bash
laconic-so --stack uniswap-urbit-app deploy create --spec-file uniswap-urbit-app-spec.yml --deployment-dir uniswap-urbit-app-deployment
```

open `uniswap-urbit-app-deployment/config.env` and set the following

```bash
# App to be installed (Do not change)
CERC_URBIT_APP=uniswap

# External RPC endpoints
# https://docs.infura.io/getting-started#2-create-an-api-key
# not required for demo
CERC_INFURA_KEY=

# Uniswap API GQL Endpoint
# Set this to GQL proxy server endpoint for uniswap app
# (Eg. http://localhost:4000/v1/graphql - in case stack is being run locally with proxy enabled)
# (Eg. https://abc.xyz.com/v1/graphql - in case https://abc.xyz.com is pointed to the proxy endpoint)
# replace `localhost` with the IP of your Digital Ocean droplet
CERC_UNISWAP_GQL=http://localhost:4000/v1/graphql

# Optional

# Whether to enable app installation on Urbit
# (just builds and uploads the glob file if disabled) (Default: true)
CERC_ENABLE_APP_INSTALL=

# Whether to run the proxy GQL server
# (disable only if proxy not required to be run) (Default: true)
CERC_ENABLE_PROXY=

# Proxy server configuration
# Used only if proxy is enabled

# Upstream API URL
# (Eg. https://api.example.org)
CERC_PROXY_UPSTREAM=https://api.uniswap.org

# Origin header to be used in the proxy
# (Eg. https://app.example.org)
CERC_PROXY_ORIGIN_HEADER=https://app.uniswap.org

# IPFS configuration

# IFPS endpoint to host the glob file on
# (Default: http://ipfs:5001 pointing to in-stack IPFS node)
CERC_IPFS_GLOB_HOST_ENDPOINT=

# IFPS endpoint to fetch the glob file from
# (Default: http://ipfs:8080 pointing to in-stack IPFS node)
CERC_IPFS_SERVER_ENDPOINT=
```

Great, you can now start the stack with:

```bash
laconic-so deployment --dir uniswap-urbit-app-deployment start
```

It will take awhile (5-15 mins) to deploy, you can see progress with the following command:

```
laconic-so deployment --dir uniswap-urbit-app-deployment logs -f
```

See the [status](#status) below for details to confirm correct operation. Meanwhile, let's take a look at what is happening under the hood.

For example, the `docker-compose.yml` for the fakezod that is about to be deployed looks like:

```
version: '3.7'

services:
  # Runs an Urbit fake ship and attempts an app installation using given data
  # Uploads the app glob to given IPFS endpoint
  # From urbit_app_builds volume:
  # - takes app build from ${CERC_URBIT_APP}/build (waits for it to appear)
  # - takes additional mark files from ${CERC_URBIT_APP}/mar
  # - takes the docket file from ${CERC_URBIT_APP}/desk.docket-0
  urbit-fake-ship:
    restart: unless-stopped
    image: tloncorp/vere
    environment:
      CERC_SCRIPT_DEBUG: ${CERC_SCRIPT_DEBUG}
      CERC_URBIT_APP: ${CERC_URBIT_APP}
      CERC_ENABLE_APP_INSTALL: ${CERC_ENABLE_APP_INSTALL:-true}
      CERC_IPFS_GLOB_HOST_ENDPOINT: ${CERC_IPFS_GLOB_HOST_ENDPOINT:-http://ipfs:5001}
      CERC_IPFS_SERVER_ENDPOINT: ${CERC_IPFS_SERVER_ENDPOINT:-http://ipfs:8080}
    entrypoint: ["bash", "-c", "./run-urbit-ship.sh && ./deploy-app.sh && tail -f /dev/null"]
    volumes:
      - urbit_data:/urbit
      - urbit_app_builds:/app-builds
      - ../config/urbit/run-urbit-ship.sh:/urbit/run-urbit-ship.sh
      - ../config/urbit/deploy-app.sh:/urbit/deploy-app.sh
    ports:
      - "80"
    healthcheck:
      test: ["CMD", "nc", "-v", "localhost", "80"]
      interval: 20s
      timeout: 5s
      retries: 15
      start_period: 10s

volumes:
  urbit_data:
  urbit_app_builds:
```

On deploy of the above, a fakezod will be deployed and wait for the static build to be globbed. The glob file will be published to a locally running IPFS node, which will be referenced when updating the `desk.docket-0` file. Respectively, the scripts `run-urbit-ship.sh` and `deploy-app.sh` look like:

```
#!/bin/bash

pier_dir="/urbit/zod"

# Run urbit ship in daemon mode
# Check if the directory exists
if [ -d "$pier_dir" ]; then
  echo "Pier directory already exists, rebooting..."
  /urbit/zod/.run -d
else
  echo "Creating a new fake ship..."
  urbit -d -F zod
fi
```

and

```
#!/bin/bash

if [ -z "$CERC_URBIT_APP" ]; then
  echo "CERC_URBIT_APP not set, exiting"
  exit 0
fi

echo "Creating Urbit application for ${CERC_URBIT_APP}"

app_desk_dir=/urbit/zod/${CERC_URBIT_APP}
if [ -d ${app_desk_dir} ]; then
  echo "Desk dir already exists for ${CERC_URBIT_APP}, skipping deployment..."
  exit 0
fi

app_build=/app-builds/${CERC_URBIT_APP}/build
app_mark_files=/app-builds/${CERC_URBIT_APP}/mar
app_docket_file=/app-builds/${CERC_URBIT_APP}/desk.docket-0

echo "Reading app build from ${app_build}"
echo "Reading additional mark files from ${app_mark_files}"
echo "Reading docket file ${app_docket_file}"

# Loop until the app's build appears
while [ ! -d ${app_build} ]; do
  echo "${CERC_URBIT_APP} app build not found, retrying in 5s..."
  sleep 5
done
echo "Build found..."

echo "Using IPFS endpoint ${CERC_IPFS_GLOB_HOST_ENDPOINT} for hosting the ${CERC_URBIT_APP} glob"
echo "Using IPFS server endpoint ${CERC_IPFS_SERVER_ENDPOINT} for reading ${CERC_URBIT_APP} glob"
ipfs_host_endpoint=${CERC_IPFS_GLOB_HOST_ENDPOINT}
ipfs_server_endpoint=${CERC_IPFS_SERVER_ENDPOINT}

# Fire curl requests to perform operations on the ship
dojo () {
  curl -s --data '{"source":{"dojo":"'"$1"'"},"sink":{"stdout":null}}' http://localhost:12321
}

hood () {
  curl -s --data '{"source":{"dojo":"+hood/'"$1"'"},"sink":{"app":"hood"}}' http://localhost:12321
}

# Create / mount the app's desk
hood "merge %${CERC_URBIT_APP} our %landscape"
hood "mount %${CERC_URBIT_APP}"

# Copy over build to desk data dir
cp -r ${app_build} ${app_desk_dir}

# Copy over the additional mark files (if required for your application)
cp ${app_mark_files}/* ${app_desk_dir}/mar/

# Remove unnecessary files
rm "${app_desk_dir}/desk.bill"
rm "${app_desk_dir}/desk.ship"

# Commit changes and create a glob
hood "commit %${CERC_URBIT_APP}"
dojo "-landscape!make-glob %${CERC_URBIT_APP} /build"

glob_file=$(ls -1 -c zod/.urb/put | head -1)
echo "Created glob file: ${glob_file}"

# Upload the glob file to IPFS (running locally by default)
echo "Uploading glob file to ${ipfs_host_endpoint}"
upload_response=$(curl -X POST -F file=@./zod/.urb/put/${glob_file} ${ipfs_host_endpoint}/api/v0/add)
glob_cid=$(echo "$upload_response" | grep -o '"Hash":"[^"]*' | sed 's/"Hash":"//')

glob_url="${ipfs_server_endpoint}/ipfs/${glob_cid}?filename=${glob_file}"
glob_hash=$(echo "$glob_file" | sed "s/glob-\([a-z0-9\.]*\).glob/\1/")

echo "Glob file uploaded to IFPS:"
echo "{ cid: ${glob_cid}, filename: ${glob_file} }"
echo "{ url: ${glob_url}, hash: ${glob_hash} }"

# Exit if the installation not required
if [ "$CERC_ENABLE_APP_INSTALL" = "false" ]; then
  echo "CERC_ENABLE_APP_INSTALL set to false, skipping app installation"
  exit 0
fi

# Curl and wait for the glob to be hosted
echo "Checking if glob file hosted at ${glob_url}"
while true; do
  response=$(curl -sL -w "%{http_code}" -o /dev/null "$glob_url")

  if [ $response -eq 200 ]; then
    echo "File found at $glob_url"
    break  # Exit the loop if the file is found
  else
    echo "File not found, retrying in a 5s..."
    sleep 5
  fi
done

# Copy in the docket file and substitute the glob URL and hash
cp ${app_docket_file} ${app_desk_dir}/
sed -i "s|REPLACE_WITH_GLOB_URL|${glob_url}|g; s|REPLACE_WITH_GLOB_HASH|${glob_hash}|g" ${app_desk_dir}/desk.docket-0

# Commit changes and install the app
hood "commit %${CERC_URBIT_APP}"
hood "install our %${CERC_URBIT_APP}"

echo "${CERC_URBIT_APP} app installed"
```

Thus, once we start the stack, a loop waits for the static build to complete and the glob to be published, then finalizes installating of the application with the updated `desk.docket-0`.

The `docker-compose.yml` for the uniswap-interface looks like:

```
version: "3.2"

services:
  uniswap-interface:
    image: cerc/uniswap-interface:local
    restart: on-failure
    environment:
      - REACT_APP_INFURA_KEY=${CERC_INFURA_KEY}
      - REACT_APP_AWS_API_ENDPOINT=${CERC_UNISWAP_GQL}
    command: ["./build-app.sh"]
    volumes:
      - ../config/uniswap-interface/build-app.sh:/app/build-app.sh
      - urbit_app_builds:/app-builds
      - ../config/uniswap-interface/urbit-files/mar:/app/mar
      - ../config/uniswap-interface/urbit-files/desk.docket-0:/app/desk.docket-0

volumes:
  urbit_app_builds:
```

whereby `build-app.sh` looks like:

```
#!/bin/bash

# Check and exit if a deployment already exists (for restarts)
if [ -d /app-builds/uniswap/build ]; then
  echo "Build already exists, remove volume to rebuild"
  exit 0
fi

yarn build

# Copy over build and other files to app-builds for urbit deployment
mkdir -p /app-builds/uniswap
cp -r ./build /app-builds/uniswap/

cp -r mar /app-builds/uniswap/
cp desk.docket-0 /app-builds/uniswap/
```

Throughout the process, `docker volumes` are used for availability of files across docker containers.

There are three mark files that we needed to create and include (recall that any file extensions in the static build that the %landscape desk does not contain need to be added to your the desk for); `map.hoon`, `ttf.hoon`, and `woff.hoon`. See them [here](https://github.com/cerc-io/stack-orchestrator/tree/main/stack_orchestrator/data/config/uniswap-interface/urbit-files/mar) and this is what `woff.hoon` looks like:

```
|_  dat=octs
++  grow
  |%
  ++  mime  [/font/woff dat]
  --
++  grab
  |%
  ++  mime  |=([=mite =octs] octs)
  ++  noun  octs
  --
++  grad  %mime
--
```

The `desk.docket-0` file is application specific; for Uniswap it looks like:

```
:~  title+'Uniswap'
    info+'Self-hosted uniswap frontend.'
    color+0xcd.75df
    image+'https://logowik.com/content/uploads/images/uniswap-uni7403.jpg'
    base+'uniswap'
    glob-http+['REPLACE_WITH_GLOB_URL' REPLACE_WITH_GLOB_HASH]
    version+[0 0 1]
    website+'https://uniswap.org/'
    license+'MIT'
==
```

Recall from a script above, we use `sed` to populate the `glob-http` field. With all these pieces in place, we can start the stack...

### Status

Depending on the specs of your machine, starting a deployment can take anywhere from 5-15 minutes.

Recall that you can run the following to view logs from all processes as they come in:

```
laconic-so deployment --dir uniswap-urbit-app-deployment logs -f
```

Eventually, you'll see:

```
laconic-3ccf7ee79bdae874-urbit-fake-ship-1    | docket: fetching %http glob for %uniswap desk
laconic-3ccf7ee79bdae874-urbit-fake-ship-1    | ">="">="uniswap app installed
```

which is good. Exit from following those logs then double check that everything is by running `docker ps`, all containers should be `healthy`.

Fakezod's have the same default password of `lidlut-tabwed-pillex-ridrup` and you can confirm this by running the following command:

```
laconic-so deployment --dir uniswap-urbit-app-deployment exec urbit-fake-ship "curl -s --data '{\"source\":{\"dojo\":\"+code\"},\"sink\":{\"stdout\":null}}' http://localhost:12321"
```

Navigate to http://localhost:8080 and enter the password to login. You should see the Uniswap tile. If you have MetaMask installed in your browser, Uniswap should work.

## Automate

After this step, we use this pair of scripts to publish the desk to a live planet. You can do the same for your application and add it to CI workflows in order to publish the latest version of your app to your Urbit ship.

```
#!/bin/bash

# $1: Remote user host
# $2: App name (eg. uniswap)
# $3: Assets dir path (local) for app (eg. /home/user/myapp/urbit-files)
# $4: Remote Urbit ship's pier dir path (eg. /home/user/zod)
# $5: Glob file URL (eg. https://xyz.com/glob-0vabcd.glob)
# $6: Glob file hash (eg. 0vabcd)

if [ "$#" -ne 6 ]; then
  echo "Incorrect number of arguments"
  echo "Usage: $0 <username@remote_host> <app_name> </path/to/app/assets/folder> </path/to/remote/pier/folder> <glob_url> <glob_hash>"
  exit 1
fi

remote_user_host="$1"
app_name=$2
app_assets_folder=$3
remote_pier_folder="$4"
glob_url="$5"
glob_hash="$6"

installation_script="./install-urbit-app.sh"

# Copy over the assets to remote machine in a tmp dir
remote_app_assets_folder=/tmp/urbit-app-assets/$app_name
ssh "$remote_user_host" "mkdir -p $remote_app_assets_folder"
scp -r $app_assets_folder/* $remote_user_host:$remote_app_assets_folder

# Run the installation script
ssh "$remote_user_host" "bash -s $app_name $remote_app_assets_folder '${glob_url}' $glob_hash $remote_pier_folder" < "$installation_script"

# Remove the tmp assets dir
ssh "$remote_user_host" "rm -rf $remote_app_assets_folder"
```

the `./install-urbit-app.sh` looks like:

```
#!/bin/bash

# $1: App name (eg. uniswap)
# $2: Assets dir path (local) for app (eg. /home/user/myapp/urbit-files)
# $3: Glob file URL (eg. https://xyz.com/glob-0vabcd.glob)
# $4: Glob file hash (eg. 0vabcd)
# $5: Urbit ship's pier dir (default: ./zod)

if [ "$#" -lt 4 ]; then
  echo "Insufficient arguments"
  echo "Usage: $0 <app_name> </path/to/app/assets/folder> <glob_url> <glob_hash> [/path/to/remote/pier/folder]"
  exit 1
fi

app_name=$1
app_mark_files=$2/mar
app_docket_file=$2/desk.docket-0
echo "Creating Urbit application for ${app_name}"
echo "Reading additional mark files from ${app_mark_files}"
echo "Reading docket file ${app_docket_file}"

glob_url=$3
glob_hash=$4
echo "Using glob file from ${glob_url} with hash ${glob_hash}"

# Default pier dir: ./zod
# Default desk dir: ./zod/<app_name>
pier_dir="${5:-./zod}"
app_desk_dir="${pier_dir}/${app_name}"
echo "Using ${app_desk_dir} as the ${app_name} desk dir path"

# Fire curl requests to perform operations on the ship
hood () {
  curl -s --data '{"source":{"dojo":"+hood/'"$1"'"},"sink":{"app":"hood"}}' http://localhost:12321
}

# Create / mount the app's desk
hood "merge %${app_name} our %landscape"
hood "mount %${app_name}"

# Copy over the additional mark files
cp ${app_mark_files}/* ${app_desk_dir}/mar/

rm "${app_desk_dir}/desk.bill"
rm "${app_desk_dir}/desk.ship"

# Replace the docket file for app
# Substitue the glob URL and hash
cp ${app_docket_file} ${app_desk_dir}/
sed -i "s|REPLACE_WITH_GLOB_URL|${glob_url}|g; s|REPLACE_WITH_GLOB_HASH|${glob_hash}|g" ${app_desk_dir}/desk.docket-0

# Commit changes and install the app
hood "commit %${app_name}"
hood "install our %${app_name}"

echo "${app_name} app installed"
```

with the main difference here being that the glob already exists and you tell the script which glob to install from. You could imagine hosting a ship with multiple versions of your application.

## Summary & Next steps

## References

- https://github.com/cerc-io/stack-orchestrator/blob/main/stack_orchestrator/data/stacks/uniswap-urbit-app/README.md
- https://github.com/cerc-io/stack-orchestrator/tree/main/stack_orchestrator/data/config/urbit

## Glossary

ship - a running Urbit
desk - "app" installed on a ship
planet - an UrbitID that runs as a ship
glob file - output after feedind glob files to the globulator
mark file - Urbit apps need a mark file for every file extension
landscape - a default app (desk) that comes with *most* mark files that you need.
Hoon - a programming language for Urbit; not relevant to this guide
