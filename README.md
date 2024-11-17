# 프론트엔드 배포 파이프라인 가이드

## 개요

![aws drawio](https://github.com/user-attachments/assets/9f10c44c-6538-484a-aed9-b6cc4e053ad9)

GitHub Actions에 워크플로우를 작성해 다음과 같이 배포가 진행되도록 합니다:

## GitHub Actions 워크플로우 작성 가이드

프로젝트 루트의 `.github/workflows/deploy.yml` 파일에 다음 내용을 작성합니다:

### 워크플로우 이름 설정

```yaml
name: Deploy Next.js to S3 and invalidate CloudFront
```

- GitHub Actions 대시보드에 표시될 워크플로우의 이름을 정의합니다.

### 트리거 조건 설정

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- `push`: main 브랜치에 코드가 푸시될 때 자동으로 실행됩니다.
- `workflow_dispatch`: GitHub UI에서 수동으로 워크플로우를 실행할 수 있습니다.

### 작업 환경 설정

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

- `jobs`: 실행할 작업들을 정의합니다.
- `deploy`: 작업의 이름입니다.
- `runs-on`: 최신 우분투 환경에서 작업이 실행됩니다.

### 실행 단계 설정

```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v2
```

- 소스 코드를 가져오는 첫 단계입니다.

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v2
  with:
    node-version: "18.x"
```

- Node.js 18.x 버전을 설정합니다.

```yaml
- name: Install dependencies
  run: npm ci
```

- `npm ci` 명령어로 프로젝트 의존성을 설치합니다.
- `npm install`보다 더 엄격하고 일관된 설치를 보장합니다.

```yaml
- name: Build
  run: npm run build
```

- Next.js 프로젝트를 빌드합니다.
- 결과물은 `out` 디렉토리에 생성됩니다.

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v1
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ secrets.AWS_REGION }}
```

- AWS 인증 정보를 설정합니다.
- GitHub Secrets에 저장된 AWS 접근 키, 시크릿 키, 리전 정보를 사용합니다.

```yaml
- name: Deploy to S3
  run: |
    aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete
```

- 빌드된 파일들을 S3 버킷에 동기화합니다.
- `--delete` 옵션은 S3에는 있지만 로컬에는 없는 파일들을 삭제합니다.

```yaml
- name: Invalidate CloudFront cache
  run: |
    aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

- CloudFront 캐시를 무효화합니다.
- `/*`는 모든 경로의 캐시를 무효화한다는 의미입니다.
- 사용자들이 항상 최신 버전의 웹사이트를 볼 수 있도록 합니다.

## 전체 워크플로우 파일

다음은 전체 `deploy.yml` 파일의 내용입니다:

```yaml
name: Deploy Next.js to S3 and invalidate CloudFront

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: "18.x"

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Deploy to S3
        run: |
          aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

## GitHub Secrets 설정

워크플로우 실행을 위해 GitHub 저장소의 Settings > Secrets and variables > Actions 메뉴에서 다음 값들을 설정해야 합니다:

1. `AWS_ACCESS_KEY_ID`: AWS IAM 사용자의 액세스 키 ID
2. `AWS_SECRET_ACCESS_KEY`: AWS IAM 사용자의 시크릿 액세스 키
3. `AWS_REGION`: AWS 리전 (예: ap-northeast-2)
4. `S3_BUCKET_NAME`: 정적 웹사이트 호스팅용 S3 버킷 이름
5. `CLOUDFRONT_DISTRIBUTION_ID`: CloudFront 배포 ID

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: http://subin-s3-test.s3-website-ap-southeast-2.amazonaws.com/
- CloudFrount 배포 도메인 이름: https://d2icfx053w3v0p.cloudfront.net/

## 주요 개념

- **GitHub Actions**과 CI/CD 도구: GitHub에서 제공하는 자동화된 워크플로우 도구입니다. 코드 변경사항이 발생할 때마다 자동으로 빌드, 테스트, 배포 등의 작업을 수행하여 지속적 통합(CI)과 지속적 배포(CD)를 가능하게 합니다.
- **S3와 스토리지**: Amazon S3(Simple Storage Service)는 AWS에서 제공하는 클라우드 스토리지 서비스입니다. 정적 웹사이트 호스팅이 가능하며, 높은 내구성과 가용성을 제공합니다. 이 프로젝트에서는 빌드된 Next.js 애플리케이션의 정적 파일들을 저장하고 서비스하는 데 사용됩니다.
- **CloudFront와 CDN**: Amazon CloudFront는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스입니다. 전 세계에 분산된 엣지 로케이션을 통해 콘텐츠를 캐싱하고 더 빠른 속도로 사용자에게 전달합니다. S3에 저장된 정적 파일들을 전 세계 사용자들에게 빠르게 제공하는 역할을 합니다.
- **캐시 무효화(Cache Invalidation)**: CloudFront가 엣지 로케이션에 캐싱한 콘텐츠를 강제로 새로고침하는 프로세스입니다. 새로운 버전의 웹사이트가 배포될 때 실행되어, 사용자들이 항상 최신 버전의 콘텐츠를 받아볼 수 있도록 보장합니다.
- **Repository secret과 환경변수**: GitHub 저장소에서 안전하게 보관하는 민감한 정보들입니다. AWS 인증 정보와 같은 중요한 값들을 코드에 직접 노출시키지 않고 안전하게 사용할 수 있게 해줍니다. 이 프로젝트에서는 AWS 접근 키, 시크릿 키, 리전 정보 등을 저장하는데 사용됩니다.

## 개발 환경 설정

이 프로젝트는 `create-next-app`으로 생성된 Next.js 프로젝트입니다.

### 개발 서버 실행

```bash
npm run dev
# or
yarn dev
```
