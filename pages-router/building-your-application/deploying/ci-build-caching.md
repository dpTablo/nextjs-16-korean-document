# CI 빌드 캐싱

Next.js는 빌드 성능 개선을 위해 `.next/cache`에 캐시를 저장합니다. CI 환경에서 빌드 간 캐시를 유지하도록 설정하면 빌드 시간을 크게 줄일 수 있습니다.

## 캐시 디렉토리

`.next/cache` 디렉토리에는 다음 항목이 저장됩니다:

- 컴파일된 JavaScript 및 CSS
- 최적화된 이미지
- 빌드 메타데이터

## CI 제공자별 설정

### Vercel

Vercel에서는 자동으로 캐싱이 설정됩니다. 추가 조치가 필요하지 않습니다.

### GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            ${{ github.workspace }}/.next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-

      - run: npm ci
      - run: npm run build
```

### CircleCI

```yaml
# .circleci/config.yml
version: 2.1

jobs:
  build:
    docker:
      - image: cimg/node:20.0
    steps:
      - checkout
      - restore_cache:
          keys:
            - dependency-cache-{{ checksum "yarn.lock" }}
      - run: yarn install
      - save_cache:
          key: dependency-cache-{{ checksum "yarn.lock" }}
          paths:
            - ./node_modules
            - ./.next/cache
      - run: yarn build
```

### Travis CI

```yaml
# .travis.yml
language: node_js
node_js:
  - 20

cache:
  directories:
    - $HOME/.cache/yarn
    - node_modules
    - .next/cache

script:
  - yarn install
  - yarn build
```

### GitLab CI

```yaml
# .gitlab-ci.yml
image: node:20

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .next/cache/

build:
  stage: build
  script:
    - npm ci
    - npm run build
```

### Netlify

Netlify Plugins를 사용합니다. `@netlify/plugin-nextjs`를 설치하면 자동으로 캐싱이 처리됩니다.

```toml
# netlify.toml
[[plugins]]
  package = "@netlify/plugin-nextjs"
```

### AWS CodeBuild

```yaml
# buildspec.yml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci
  build:
    commands:
      - npm run build

cache:
  paths:
    - 'node_modules/**/*'
    - '.next/cache/**/*'
```

### Bitbucket Pipelines

```yaml
# bitbucket-pipelines.yml
image: node:20

definitions:
  caches:
    nextcache: .next/cache

pipelines:
  default:
    - step:
        name: Build
        caches:
          - node
          - nextcache
        script:
          - npm ci
          - npm run build
```

### Heroku

`package.json`에 캐시 디렉토리를 지정합니다:

```json
{
  "cacheDirectories": [".next/cache"]
}
```

### Azure Pipelines

```yaml
# azure-pipelines.yml
pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node.js'

  - task: Cache@2
    displayName: 'Cache .next/cache'
    inputs:
      key: next | $(Agent.OS) | yarn.lock
      path: '$(System.DefaultWorkingDirectory)/.next/cache'

  - script: |
      npm ci
      npm run build
    displayName: 'Install and Build'
```

### Jenkins (Pipeline)

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                writeFile file: "next-lock.cache", text: "${GIT_COMMIT}"

                cache(caches: [
                    arbitraryFileCache(
                        path: ".next/cache",
                        includes: "**/*",
                        cacheValidityDecidingFile: "next-lock.cache"
                    )
                ]) {
                    sh "npm ci"
                    sh "npm run build"
                }
            }
        }
    }
}
```

## 캐시 키 전략

효과적인 캐시 키를 설정하면 불필요한 캐시 무효화를 방지할 수 있습니다:

```yaml
# 권장 캐시 키 전략
key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}

# 복원 키 (부분 일치)
restore-keys: |
  ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
  ${{ runner.os }}-nextjs-
```

## 문제 해결

### "No Cache Detected" 오류

캐시가 올바르게 설정되지 않은 경우 이 오류가 발생할 수 있습니다.

**확인 사항:**
1. 캐시 경로가 `.next/cache`를 포함하는지 확인
2. 캐시 키가 올바르게 설정되었는지 확인
3. 이전 빌드의 캐시가 존재하는지 확인

### 캐시가 너무 큰 경우

캐시 크기가 CI 제공자의 한도를 초과하는 경우:

1. 불필요한 파일을 `.gitignore`에 추가
2. `next.config.js`에서 캐시 설정 최적화:

```js
// next.config.js
const nextConfig = {
  generateBuildId: async () => {
    // 빌드 ID 고정으로 캐시 효율 향상
    return 'my-build-id'
  },
}
```

### 캐시 무효화

캐시를 강제로 무효화해야 하는 경우:

1. 캐시 키에 버전 번호 추가
2. CI 제공자의 캐시 삭제 기능 사용
3. `package-lock.json` 또는 `yarn.lock` 업데이트
