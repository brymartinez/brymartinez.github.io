---
title: Webpack Config for NestJS with Serverless
description: Serverless Framework-driven NestJS app deployed on Lambda.
pubDate: 2026-04-25
topic: NestJS
draft: false
---

## Notes:

- You cannot bundle using `nest build --webpack` with `serverless` because it doesn't take `<stage>-config.yml` to account
- Use `serverless-webpack` first, then `serverless-offline`
- Do not use `custom.webpack.includeModules` . `.npmrc` will not be included in `.webpack` folder for fresh `npm install`
- Use `package.excludeDevDependencies`

**Default `webpack-config.yml`**

```jsx
const path = require('path');
const webpack = require('webpack');
const slsw = require('serverless-webpack');
const ForkTSCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');

// List of Nestjs lazy imports
const lazyImports = [
  '@nestjs/microservices/microservices-module',
  '@nestjs/websockets/socket-module',
  '@nestjs/platform-express',
  '@grpc/grpc-js',
  '@grpc/proto-loader',
  'kafkajs',
  'mqtt',
  'nats',
  'ioredis',
  'amqplib',
  'amqp-connection-manager',
  'pg-native', // for `pg` only
  'cache-manager',
];

module.exports = {
  mode: slsw.lib.webpack.isLocal ? 'development' : 'production',
  devtool: 'source-map',
  entry: slsw.lib.entries,
  target: 'node',
  resolve: {
    extensions: ['.cjs', '.mjs', '.js', '.ts'],
  },
  output: {
    // required for @vendia/serverless-express
    libraryTarget: 'commonjs2',
    path: path.join(__dirname, '.webpack'),
    filename: '[name].js',
  },
  // Must remain empty to not include node_modules in the zipped version
  externals: [],
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          configFile: 'tsconfig.webpack.json',
          transpileOnly: true,
          experimentalFileCaching: true,
        },
      },
    ],
  },
  optimization: {
    minimizer: [
      // Minimizes but keeps class names for debugging
      new TerserPlugin({
        terserOptions: {
          keep_classnames: true,
        },
      }),
    ],
  },
  plugins: [
    new ForkTSCheckerWebpackPlugin(),
    // Checks if lazyImport[] is required, and ignores if not
    new webpack.IgnorePlugin({
      checkResource(resource) {
        if (lazyImports.includes(resource)) {
          try {
            require.resolve(resource);
          } catch (err) {
            return true;
          }
        }
        return false;
      },
    }),
  ],
};
```

**Default `tsconfig.webpack.json`:**

```jsx
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "lib": ["es2020"],
    "removeComments": true,
    "moduleResolution": "node",
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "module": "commonjs",
    "target": "es2020",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "baseUrl": "./",
    "resolveJsonModule": true,
    // "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "emitDecoratorMetadata": true,
    "allowSyntheticDefaultImports": true,
    "strictNullChecks": false,
    "noImplicitAny": false,
    "strictBindCallApply": false,
    "noFallthroughCasesInSwitch": false,
  },
  "include": ["./**/*.ts"],
  "exclude": [
    "node_modules/**/*",
    ".serverless/**/*",
    ".webpack/**/*",
    ".build/**/*",
    "dist/**/*",
    "test/**/*"
  ]
}
```