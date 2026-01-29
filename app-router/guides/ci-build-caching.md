---
원문: https://nextjs.org/docs/app/guides/ci-build-caching
버전: 16.1.6
---

# CI Build Caching

CI 환경에서 빌드 캐시를 구성하여 Next.js 빌드 성능을 개선하는 방법을 알아봅니다.

## 개요

Next.js는 빌드 성능을 개선하기 위해 `.next/cache`에 빌드 캐시를 저장합니다. CI 환경에서는 빌드 간에 이 캐시를 유지하도록 구성해야 "No Cache Detected" 오류를 방지할 수 있습니다.

**캐싱의 이점:**
- 빌드 시간 단축
- CI 비용 절감
- 일관된 빌드 성능

---

## CI 제공자별 구성

### Vercel

Vercel에서는 자동으로 구성됩니다. 별도의 작업이 필요하지 않습니다.

- 내장된 Next.js 캐시 지원
- Turborepo 지원 포함

---

### GitHub Actions

GitHub Actions에서 캐시를 구성하는 방법입니다.

**.github/workflows/build.yml**
```yaml
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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Cache dependencies and Next.js build
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            ${{ github.workspace }}/.next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
```

**yarn 사용 시:**
```yaml
- name: Cache dependencies and Next.js build
  uses: actions/cache@v4
  with:
    path: |
      ~/.cache/yarn
      ${{ github.workspace }}/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('**/yarn.lock') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('**/yarn.lock') }}-
```

**pnpm 사용 시:**
```yaml
- name: Get pnpm store directory
  shell: bash
  run: |
    echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV

- name: Cache dependencies and Next.js build
  uses: actions/cache@v4
  with:
    path: |
      ${{ env.STORE_PATH }}
      ${{ github.workspace }}/.next/cache
    key: ${{ runner.os }}-nextjs-${{ hashFiles('**/pnpm-lock.yaml') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
    restore-keys: |
      ${{ runner.os }}-nextjs-${{ hashFiles('**/pnpm-lock.yaml') }}-
```

---

### GitLab CI

**.gitlab-ci.yml**
```yaml
image: node:20

stages:
  - build

variables:
  npm_config_cache: "$CI_PROJECT_DIR/.npm"

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .next/cache/
    - .npm/

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - .next/
    expire_in: 1 week
```

**병렬 작업에서 캐시 공유:**
```yaml
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
    - .next/cache/
  policy: pull-push
```

---

### CircleCI

**.circleci/config.yml**
```yaml
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
            - dependency-cache-

      - run:
          name: Install dependencies
          command: yarn install --frozen-lockfile

      - run:
          name: Build
          command: yarn build

      - save_cache:
          key: dependency-cache-{{ checksum "yarn.lock" }}
          paths:
            - ./node_modules
            - ./.next/cache

workflows:
  build:
    jobs:
      - build
```

---

### Travis CI

**.travis.yml**
```yaml
language: node_js
node_js:
  - '20'

cache:
  directories:
    - $HOME/.cache/yarn
    - node_modules
    - .next/cache

install:
  - yarn install --frozen-lockfile

script:
  - yarn build
```

---

### AWS CodeBuild

**buildspec.yml**
```yaml
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

artifacts:
  files:
    - '**/*'
  base-directory: .next
```

---

### Azure Pipelines

**azure-pipelines.yml**
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node.js'

  - task: Cache@2
    displayName: 'Cache npm packages'
    inputs:
      key: 'npm | "$(Agent.OS)" | package-lock.json'
      path: $(npm_config_cache)
      restoreKeys: |
        npm | "$(Agent.OS)"

  - task: Cache@2
    displayName: 'Cache .next/cache'
    inputs:
      key: 'next | $(Agent.OS) | package-lock.json'
      path: '$(System.DefaultWorkingDirectory)/.next/cache'
      restoreKeys: |
        next | $(Agent.OS)

  - script: npm ci
    displayName: 'Install dependencies'

  - script: npm run build
    displayName: 'Build'
```

---

### Bitbucket Pipelines

**bitbucket-pipelines.yml**
```yaml
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
        artifacts:
          - .next/**

  branches:
    main:
      - step:
          name: Build and Deploy
          caches:
            - node
            - nextcache
          script:
            - npm ci
            - npm run build
          artifacts:
            - .next/**
```

---

### Netlify

[`@netlify/plugin-nextjs`](https://www.npmjs.com/package/@netlify/plugin-nextjs) 플러그인을 사용합니다.

**netlify.toml**
```toml
[build]
  command = "npm run build"
  publish = ".next"

[[plugins]]
  package = "@netlify/plugin-nextjs"
```

---

### Heroku

**package.json**에 캐시 디렉토리를 추가합니다.

**package.json**
```json
{
  "name": "my-nextjs-app",
  "cacheDirectories": [".next/cache"],
  "scripts": {
    "build": "next build",
    "start": "next start"
  }
}
```

---

### Jenkins (Pipeline)

**Jenkinsfile**
```groovy
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                writeFile file: "next-lock.cache", text: "${GIT_COMMIT}"
                cache(caches: [
                    arbitraryFileCache(
                        path: "node_modules",
                        includes: "**/*",
                        cacheValidityDecidingFile: "package-lock.json"
                    )
                ]) {
                    sh "npm ci"
                }
            }
        }

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
                    sh "npm run build"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

---

## 캐시 키 전략

효과적인 캐시 키 설계는 빌드 성능에 중요합니다.

### 기본 전략

```yaml
# 의존성 변경 시에만 캐시 무효화
key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}
```

### 세밀한 전략

```yaml
# 의존성 + 소스 코드 변경 시 캐시 무효화
key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
```

### 폴백 키 사용

```yaml
restore-keys: |
  ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
  ${{ runner.os }}-nextjs-
```

---

## 문제 해결

### "No Cache Detected" 경고

**원인:**
- CI에서 `.next/cache` 디렉토리가 유지되지 않음

**해결:**
1. CI 구성에 `.next/cache` 경로 추가
2. 캐시 키가 올바르게 설정되었는지 확인

### 캐시가 복원되지 않음

**확인 사항:**
- 캐시 키가 정확한지 확인
- 캐시 크기 제한 확인 (일부 CI는 제한이 있음)
- 캐시 만료 정책 확인

### 빌드 시간이 개선되지 않음

**가능한 원인:**
- 캐시 무효화가 너무 자주 발생
- 캐시 키가 너무 세분화됨

**해결:**
```yaml
# 더 안정적인 캐시 키 사용
key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}
```

---

## 캐시 크기 최적화

### 불필요한 파일 제외

**.gitignore**에 추가:
```
.next/cache/webpack/
```

### 선택적 캐싱

```yaml
# 필수 캐시만 저장
path: |
  .next/cache/fetch-cache/
  .next/cache/images/
```

---

## 베스트 프랙티스

### ✅ 해야 할 것

1. **의존성과 빌드 캐시 분리**
   ```yaml
   - name: Cache node modules
     uses: actions/cache@v4
     with:
       path: node_modules
       key: node-${{ hashFiles('**/package-lock.json') }}

   - name: Cache Next.js build
     uses: actions/cache@v4
     with:
       path: .next/cache
       key: next-${{ hashFiles('**/package-lock.json') }}-${{ github.sha }}
   ```

2. **폴백 키 사용**
   - 완전 일치하지 않아도 부분 캐시 활용

3. **캐시 효과 모니터링**
   - CI 로그에서 캐시 히트/미스 확인

### ❌ 피해야 할 것

1. **너무 구체적인 캐시 키**
   ```yaml
   # ❌ 나쁜 예: 모든 커밋에서 캐시 미스
   key: ${{ github.sha }}
   ```

2. **캐시 경로 누락**
   - `.next/cache` 경로를 반드시 포함

3. **캐시 크기 무시**
   - CI 제공자의 캐시 제한 확인

---

## 다음 단계

- [Memory Usage](./memory-usage.md) - 메모리 사용량 최적화
- [Self-Hosting](./self-hosting.md) - 셀프 호스팅
- [Debugging](./debugging.md) - 디버깅

---
