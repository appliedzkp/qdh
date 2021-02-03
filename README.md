![QDH Logo](https://quadratic.page/ballot-box-emoji.png)

# Quadratic Dollar Homepage

The Quadratic Dollar Homepage is a spin on the [Million Dollar Homepage](http://www.milliondollarhomepage.com/). While
it also features a space for images on a webpage, it allows users to vote on how much space each image takes up.
Moreover, it employs a quadratic and collusion-resistant voting mechanism on Ethereum called Minimal Anti-Collusion
Infrastructure (MACI) to prevent bribery and scale images quadratically.

## How to run QDH locally

Clone this repo. Install dependencies by running `yarn` or `npm install`.

```bash
git clone https://github.com/ksaitor/qdh
cd qdh
yarn  # or `npm install`
```

Copy `.env.sample` and name it `.env`.

In `.env` set values for all the missing variables, such as `MONGO_URL`, `AZURE_STORAGE_ACCOUNT_NAME`,
`AZURE_CONTAINER_NAME`, `AZURE_KEY`, `AZURE_CONNECTION_STRING`.

```bash
cp .env.sample .env
vim .env # set `MONGO_URL`, `AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_CONTAINER_NAME`, `AZURE_KEY`, `AZURE_CONNECTION_STRING`.
```

Your `.env` file should looks something like this:
```bash
NEXT_PUBLIC_MACI_ADDRESS='0x2C2B9C9a4a25e24B174f26114e8926a9f2128FE4'
NEXT_PUBLIC_POAP_ADDRESS='0x22C1f6050E56d2876009903609a2cC3fEf83B415'

NEXT_PUBLIC_STRAPI_URL='https://strapi-admin.quadratic.page'

MONGO_URL='mongodb+srv://user:password@mongodb-ip-or-dns.com/database...'

AZURE_STORAGE_ACCOUNT_NAME='qdh'
AZURE_CONTAINER_NAME='qdh-user-images'
AZURE_KEY='24f234f234f+24f243f+24f243f/24f234f234f2f24f==...'
AZURE_CONNECTION_STRING='DefaultEndpointsProtocol=https...'
```

> - If you are are looking for a free Mongo hosting, try [Mongo Atlas](https://www.mongodb.com/cloud/atlas).
> - To get Azure keys, first create a new Azure Storage account. Within it, create a new container and name it, e.g.: `qdh-user-images`. Then go to Storage account > Settings > Access keys. Copy paste Account Name, Key and Connection String from there. We'll everntually try to make this project cloud agnostic. Feel free to contribute.


Now run `yarn dev` (or `npm run dev`)

> If you are already running `yarn dev` (or `npm run dev`), make sure to kill the process and start it again. Next.js doesn't pick up `.env` changes automatically. Hence you need to restart it manually.

On the output, you should see something like this:

```bash
Loaded env from /your-project-path/qdh/.env
ready - started server on 0.0.0.0:3000, url: http://localhost:3000
```

Frontend is now accessible on http://localhost:3000

Now you need to set up and deploy MACI:

## Setting up MACI

In a separate terminal, clone MACI: https://github.com/appliedzkp/maci

Carefully follow everything in ["Local development and testing"](https://github.com/appliedzkp/maci#local-development-and-testing): bootstrap MACI repo, install Rust, build zk-SNARKs, compile contracts... everything up to the ["Demo"](https://github.com/appliedzkp/maci#demo) section.

```bash
git clone git@github.com:appliedzkp/maci.git
cd maci
npm i && npm run bootstrap && npm run build

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh  # to install Rust
cargo install zkutil --version 0.3.2 && zkutil --help

cd circuits
npm run buildBatchUpdateStateTreeSnark && npm run buildQuadVoteTallySnark

cd ../contracts
npm run compileSol
```

From the same directory (`maci/contracts`) start a Ganache instance:

```bash
npm run ganache
```

In a separate terminal, go to `maci/cli` directory and create a new MACI election:

```bash
node ./build/index.js create -d 0xc87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3 \
	-sk macisk.8715ab59a3e88a7ceec80f214ec24a95287ef2cb399a329b6964a87f85cf51c \
	-e http://localhost:8545 \
	-s 15 \
	-o 60 \
	-bm 4 \
	-bv 4
```
> You can find a detailed guide of this step and other MACI commands in the [MACI Demonstration](https://github.com/appliedzkp/maci/tree/master/cli#demonstration) docs.

Once you've deployed MACI, and created an election you should have an output like this:

```bash
MACI: 0x2C2B9C9a4a25e24B174f26114e8926a9f2128FE4
```

Now you can go to the frontend http://localhost:3000 and interact with MACI deployment.

Don't forget to connect your Metamask to your local testnet `locahost:8545` and import one of the test wallets into it.


## Setting up Admin dashboard

Setting up and running [Admin Dashboard](https://github.com/ksaitor/qdh-admin) is not mandatory, but recommended. We've based it off an open source headless
CMS, called [Strapi](https://strapi.io/)

You'll be able to find detailed instructions on how to run it in the repo's README, but we'll make a short overview here as well.

Clone the repo https://github.com/ksaitor/qdh-admin and install dependencies with `yarn` (or `npm install`)
```bash
git clone https://github.com/ksaitor/qdh-admin
cd qdh-admin
yarn  # or `npm install`
```

Copy `.env.example` and name it `.env`.

In `.env` set values for all the missing variables, such as `MONGO_URL`, `AZURE_STORAGE_ACCOUNT_NAME`,
`AZURE_CONTAINER_NAME`, `AZURE_KEY`, `AZURE_CONNECTION_STRING` with the same values as used above.

```bash
cp .env.example .env
vim .env # set `MONGO_URL`, `AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_CONTAINER_NAME`, `AZURE_KEY`, `AZURE_CONNECTION_STRING`.
```

Run `yarn develop` to start the server locally.

The api will be available at `http://localhost:1337` and the admin panel at `http://localhost:1337/admin`

You might want to update `NEXT_PUBLIC_STRAPI_URL=http://localhost:1337` in the `.env` in _qdh frontend_, so that your local frontend talks to your locally run Strapi Admin api. Don't forget to manually kill and start the frontend server. (Next.js doen't automatically pick up .env file changes.)


## Deploying QDH frontend to prod

The repo is out of the box ready for [Vercel](https://vercel.io), [Heroku](https://heroku.com) and [Dokku](https://github.com/dokku/dokku) deployments. Just set env variables (similar to what you already have in `.env`) and follow standard deployment procedures.
