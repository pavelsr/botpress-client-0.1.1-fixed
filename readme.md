# Botpress Client

Official Botpress HTTP client for TypeScript. Queries the [Botpress API](https://botpress.com/docs/api/).

⚠️ This repo is special save of v0.1.1

## Installation

```bash
npm install --save git+https://github.com/pavelsr/botpress-client-0.1.1-fixed.git # for npm

yarn add  https://github.com/pavelsr/botpress-client-0.1.1-fixed.git # for yarn

pnpm add https://github.com/pavelsr/botpress-client-0.1.1-fixed.git # for pnpm
```

For package.json:

```
"dependencies": {
    "botpress-client": "git+https://github.com/pavelsr/botpress-client-0.1.1-fixed.git"
  }
```



## Usage

```ts
import { Client } from '@botpress/client'

type ListBotsResponse = Awaited<ReturnType<Client['listBots']>>
type Bot = ListBotsResponse['bots'][number]

const main = async () => {
  const token = 'your-token'
  const workspaceId = 'your-workspace-id'
  const client = new Client({ token, workspaceId })

  const allBots: Bot[] = []
  let nextToken: string | undefined
  do {
    const resp = await client.listBots({ nextToken })
    nextToken = resp.meta.nextToken
    allBots.push(...resp.bots)
  } while (nextToken)

  console.log(allBots)
}
void main()
```

# For what I need this fork?

This is attempt to rebuild [@botpress/client](https://github.com/botpress/botpress/tree/master/packages/client) v0.1.1 with changed `defaultApiUrl` variable

I was needed this for re-compiling [botpress/inbox](https://github.com/botpress/inbox) project cause `BP_API_URL` env variable seems like not work properly

This repo could be also used as example of forking and rebuilding quite old node.js v18 module

# How did I get this code?


```bash
git clone https://github.com/botpress/botpress
cd botpress

# sha of commit founded with:
# git log -S '"version": "0.1.1"' -- packages/client/package.json
# 02ba05100546b5ad3d6c725901bf0f6a6e0b5ada

# Then create Github fork of https://github.com/botpress/botpress
# e.g. to https://github.com/pavelsr/botpress-client-0.1.1-fixed

cd ..

git clone git@github.com:pavelsr/botpress-client-0.1.1-fixed.git
cd botpress-client-0.1.1-fixed
git checkout 02ba05100546b5ad3d6c725901bf0f6a6e0b5ada
git checkout -b fix-defaultApiUrl-0.1.1

find . -mindepth 1 -maxdepth 1 -type f ! -name "tsconfig.json" ! -name '.gitignore' -exec rm -f {} +
find . -mindepth 1 -maxdepth 1 -type d ! -path "./packages" ! -path "./.git" -exec rm -rf {} +
cd packages
find . -mindepth 1 -maxdepth 1 -type d ! -name "client" -exec rm -rf {} +
cd ..
git commit -am "Remove unused files"

sed -i 's|https://api.botpress.cloud|https://botpress.linsec.dev/api|' packages/client/src/config.ts
git commit -am "Fixed defaultApiUrl"

sed -i '/^\s*\/\//d' tsconfig.json
jq -s '.[1] * .[0]' tsconfig.json packages/client/tsconfig.json > packages/client/tsconfig.merged.json
rm tsconfig.json
mv packages/client/tsconfig.merged.json packages/client/tsconfig.json

shopt -s dotglob
mv ./packages/client/* ./
rmdir ./packages/client  # rm -rf packages
git add -A
git commit -am "Fixed @botpress/client directory root"

pnpm install --save-dev ts-node @types/node @tsconfig/node18-strictest
git add -A
git commit -am "Saved build dependencies"

npm run generate
git add -A
git commit -am "Added result of npm run generate"

sed -i '/^dist$/s/^/#/' .gitignore
npm run build
git add -A
git commit -am "Added result of npm run build (dist folder)"

git checkout master
git reset --hard fix-defaultApiUrl-0.1.1
git push origin master --force
```