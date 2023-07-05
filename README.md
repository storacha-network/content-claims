# Content Claims

Implementation of the Content Claims Protocol.

Read the [spec](https://hackmd.io/@gozala/content-claims).


## Getting Started

The repo contains the infra deployment code and the service implementation.

```
├── packages   - content-claims core and lambda wrapper
└── stacks     - sst and AWS CDK code to deploy all the things
```

To work on this codebase **you need**:

- Node.js >= v18 (prod env is node v18)
- An AWS account with the AWS CLI configured locally
- Copy `.env.tpl` to `.env` and fill in the blanks
- Install the deps with `npm i`

Deploy dev services to your AWS account and start dev console. You may need to provide a `--profile` to pick the aws profile to deploy with.

```sh
npm start
```

See: https://docs.sst.dev for more info on how things get deployed.


## Environment variables

The following should be set in the env when deploying. Copy `.env.tpl` to `.env` to set in dev.

```sh
SENTRY_DSN=<your error reporting key here>

# Name of the "blocks-cars-position" table in DynamoDB
DYNAMO_TABLE=<table name here>

# Region of the DynamoDB to query
DYNAMO_REGION=us-west-2

# (optional) CSV of S3 regions of buckets that CAR files are stored in
S3_REGIONS=us-east-1,us-east-2,us-west-2
```


## Notes

These are the types of claim that we're interested in from the spec:

### Location claim

```js
{
  "op": "assert/location",
  "rsc": "https://web3.storage",
  "input": {
    "content": CID /* CAR CID */, 
    "location": [ "https://r2.cf/bag...car", "s3://bucket/bag...car" ],
    "range": { "offset": 0 } /* Optional: Byte Range in URL */
  }
}
```
we need to generate this?

### Inclusion claim

Claims that within a CAR, there are a bunch of CIDs (and their offsets).

```js
{
  "op": "assert/inclusion",
  "rsc": "https://web3.storage",
  "input": {
    "content":  CID /* CAR CID */,
    "includes": CID /* CARv2 Index CID */,
    "proof":    CID /* Optional: zero-knowledge proof */
  }
}
```
index included in invocation?

### Partition claim

Claims that a DAG can be found in a bunch of CAR files.

```js
{
  "op": "assert/partition",
  "rsc": "https://web3.storage",
  "input": {
    "content": CID /* Content Root CID */,
    "blocks": CID, /* CIDs CID */
    "parts": [
      CID /* CAR CID */,
      CID /* CAR CID */,
      ...
    ]
  }
}
```

### Relation claim 🆕

Claims that a block of content links to other blocks.

```js
{
  "op": "assert/relation",
  "rsc": "https://web3.storage",
  "input": {
    "parent": CID /* Block CID */,
    "child": [
      CID /* Linked block CID */,
      CID /* Linked block CID */,
      ...
    ]
  }
}
```

===

Dynamo tables:

## block-links

Partition key: block multihash
Sort key: part CID

```ts
interface Item {
  multihash: string
  part: string
  offset: number
  length: number
  links: Array<{ multihash: string, part: string, offset: number, length: number }>
}
```
