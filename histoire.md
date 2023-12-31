# Setup Histoire for Nuxt3

## Environment

以下 2 つのライブラリを導入する必要あり。

- [google/zx](https://github.com/google/zx)
- [jq](https://github.com/jqlang/jq)

```
# zxが存在しない場合はインストール、存在する場合はバージョンを表示する
if [ -z `which zx` ]; then
  echo 'zx is not installed'
  echo 'installing zx'
  npm i -g zx
  echo 'zx installed'
else
  zx --version
fi

# jqが存在しない場合はインストール、存在する場合はバージョンを表示する
if [ -z `which jq` ]; then
  echo 'jq is not installed'
  echo 'installing jq'
  brew install jq
  echo 'jq installed'
else
  jq --version
fi
```

## Installation

Show [Getting started with Histoire#Installation](https://histoire.dev/guide/vue3/getting-started.html#installation).

```sh
npm i -D histoire @histoire/plugin-vue @histoire/plugin-nuxt
```

## Setup

```js
// setup histoire config for Nuxt
const promise1 = $`cat <<EOF > histoire.config.ts
import { HstNuxt } from "@histoire/plugin-nuxt";
import { HstVue } from "@histoire/plugin-vue";
import { defineConfig } from "histoire";

export default defineConfig({
  plugins: [HstVue(), HstNuxt()],
});
EOF`;

// setup env.d.ts for TypeScript
const promise2 = $`cat <<EOF > env.d.ts
/// <reference types="@histoire/plugin-vue/components" />
EOF`;

/** @type {string} */
const TSCONFIG = "tsconfig.json";
const promise3 = $`mv ${TSCONFIG} _${TSCONFIG} \
&& cat _${TSCONFIG} | jq '.include |= . + ["env.d.ts", "src/**/*", "src/**/*.vue"]' > ${TSCONFIG} \
&& rm -f _${TSCONFIG}`;

/** @type {string} */
const PACKAGE = "package.json";
const promise4 = $`mv ${PACKAGE} _${PACKAGE} \
&& cat _${PACKAGE} \
| jq '.scripts |= . + { "story:dev": "histoire dev", "story:build": "histoire build", "story:prev": "histoire preview" }' > ${PACKAGE} \
&& rm -f _${PACKAGE}`;

await Promise.all([promise1, promise2, promise3, promise4]);
```

### Setup histoire.setup.ts for Pinia

```js
const isCreate = await question(
  "Create histoire.setup.ts for 'Pinia'? (y/n) > ",
);
if (isCreate === "y") {
  await $`cat <<EOF > histoire.setup.ts
import { defineSetupVue3 } from "@histoire/plugin-vue";
import { createPinia } from "pinia";

export const setupVue3 = defineSetupVue3(({ app }) => {
  const pinia = createPinia();
  app.use(pinia);
});
EOF`;
}
```
