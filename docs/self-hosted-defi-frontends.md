# Self Hosted DeFi Front Ends

TL;DR: `install ~lanfyn-dasnys %uniswap`

Applications in web3 still remain significantly centralized. Usually, the blockchain used by a particular application cannot easily be censored. However, front ends are always served to users via a centralized pipeline of service providers, each of which is a choke point for that application.

On Urbit, an application you install on your ship is always available to you. It will be available to others if you publish it to your ship. However, it will only remain available to others if your ship is online. What was once an easy to install application (for others) can easily disappear. Indeed, this is what happened to the Osmosis and Uniswap front ends on Urbit.

We address these issues by documenting the pipeline for DeFi applications to publish their front ends on Urbit. Broadly, the steps are as such:

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

Your app front end must comply with a variety of Urbit requirements. The first step is to ensure that you can compile a static build of your application. This build will be consumed by the `globulator` and these files will need to be located in the root of your ships directory. More on this later. You might already have `yarn build:static` or an existing way to generate a static build for your app. If not, ensure your application can run as intended when served as a static build.

The next step is to ensure a few things about the paths/URL in your app. First, all the uppercase letters in all the filenames of your app need to be converted to lowercase. As well, ensure that the file paths and URL does not contain square brackets. Next.js can do awkward things in this regard, for example, with the dynamic path rendering in Osmosis.

Next, you’ll need to generate mark files that are missing from the default %landscape desk. For simple applications, this should not be necessary. Every file extension in your application requires a corresponding mark file. These are short files written in Hoon that are required for the globulator to function correctly. By inspecting existing mark files, you should get a good idea as to what a new one needs to look like if your application contains any exotic file extensions.

Finally, you’ll have to decide what to do with external API calls and other services that your app uses. In the case of Uniswap and Osmosis, we run a proxy server and forward requests to the original application. Other solutions to address this issue are outside the scope of this blog post.

You can view the modifications required for each Uniswap and Osmosis. These changes aren’t upstreamed, therefore publishing new versions to Urbit requires manually rebasing and addressing any merge conflicts, followed by re-globbing and updating the `desk.docket-0` file, as described next.

## Globulate

The majority of any Urbit app is packaged up into something called a `glob`. This glob can either be served over http or ames. Most of the Urbit documentation focuses on examples using ames and the globulator UI, rather than http over the command line. For automation, we use the latter.

The static build of the application are the files that need to be globbed. A directory of these files needs to be located in the root of your ships directory. Under the hood, we’ve abstracted away the majority of this part.

### Glob Hosting

An http glob can be hosted wherever you want, like Amazon S3 or Digital Ocean Spaces Object Storage. The Laconic solution includes by default a locally running ipfs node to which the glob is published.

### desk.docket-0

The tile for each app that you see when logging into your Urbit is defined by the desk.docket-0. Therefore, the 

## Install and Publish

This part is ensuring that your desk.docket-0 is correct then running `:install our %uniswap` in the Urbit dojo. The app should now be available as a tile in Landscape. To make it available for others to install,run `:treaty publish %uniswap`

To separate the development and production lifecycles, we use a fakezod for reviewing and testing staging deployments, then when ready to publish to the network, repurpose a deployment script directed at a live planet (~lanfyn-dasnys).

## Demo

With a handful of new concepts involved in Urbit app development, automated DeFi deployments happened to be a great fit for the Laconic Stack Orchestrator tool. We’ve distilled the above steps into a few commands that can be run by anyone on a stock Digital Ocean. The following instructions will build and deploy the Uniswap front end to a fakezod.

```
laconic-so setup
laconic-so build
laconic-so deploy
```
https://github.com/cerc-io/stack-orchestrator/blob/main/stack_orchestrator/data/stacks/uniswap-urbit-app/README.md

Let’s break down what each step did and the files involved in making it happen.

### Setup

Clone the app that is modified to be built statically.

### Build

The required docker images for the app and associated service. Mainly, this should be a relatively simple `yarn build:static` added to a stock image.
// show a bunch of code here
// setup fakezod
// adding mark files
// running globulator
// install and publish

### Config

Set some configurations and environment variables

### Deploy

Using docker-compose

## Automate

Run the following script, using information relevant to your application and live planet.

```
https://github.com/cerc-io/stack-orchestrator/tree/main/stack_orchestrator/data/config/urbit

```
