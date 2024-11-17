# Next.js 자동 배포 CI/CD 파이프라인 구축

> Last Updated: 2024.03.17

## 목차

1. [소개](#소개)
2. [주요 링크 및 개념](#주요-링크-및-개념)
3. [배포 아키텍처](#배포-아키텍처)
4. [배포 파이프라인 가이드](#배포-파이프라인-가이드)
5. [GitHub Actions 워크플로우 작성 가이드](#github-actions-워크플로우-작성-가이드)
6. [GitHub Secrets 설정](#github-secrets-설정)
7. [CDN과 성능 최적화](#cdn과-성능-최적화)
8. [개발 환경 설정](#개발-환경-설정)

## 소개

이 프로젝트는 AWS S3와 CloudFront를 활용하여 Next.js 애플리케이션의 자동 배포를 위한 CI/CD 파이프라인 구축 방법을 다룹니다. 코드 변경 시 자동으로 빌드 및 배포가 이루어져 빠르고 안정적인 배포 환경을 제공합니다.

## 주요 링크 및 개념

### 주요 링크

- S3 버킷 웹사이트 엔드포인트: [http://subin-s3-test.s3-website-ap-southeast-2.amazonaws.com/](http://subin-s3-test.s3-website-ap-southeast-2.amazonaws.com/)
- CloudFrount 배포 도메인 이름: [https://d2icfx053w3v0p.cloudfront.net/](https://d2icfx053w3v0p.cloudfront.net/)

### 주요 개념

- **GitHub Actions**과 CI/CD 도구: GitHub에서 제공하는 자동화된 워크플로우 도구입니다. 코드 변경사항이 발생할 때마다 자동으로 빌드, 테스트, 배포 등의 작업을 수행하여 지속적 통합(CI)과 지속적 배포(CD)를 가능하게 합니다.

- **S3와 스토리지**: Amazon S3(Simple Storage Service)는 AWS에서 제공하는 클라우드 스토리지 서비스입니다. 정적 웹사이트 호스팅이 가능하며, 높은 내구성과 가용성을 제공합니다. 이 프로젝트에서는 빌드된 Next.js 애플리케이션의 정적 파일들을 저장하고 서비스하는 데 사용됩니다.

- **CloudFront와 CDN**: Amazon CloudFront는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스입니다. 전 세계에 분산된 엣지 로케이션을 통해 콘텐츠를 캐싱하고 더 빠른 속도로 사용자에게 전달합니다. S3에 저장된 정적 파일들을 전 세계 사용자들에게 빠르게 제공하는 역할을 합니다.

- **캐시 무효화(Cache Invalidation)**: CloudFront가 엣지 로케이션에 캐싱한 콘텐츠를 강제로 새로고침하는 프로세스입니다. 새로운 버전의 웹사이트가 배포될 때 실행되어, 사용자들이 항상 최신 버전의 콘텐츠를 받아볼 수 있도록 보장합니다.

- **Repository secret과 환경변수**: GitHub 저장소에서 안전하게 보관하는 민감한 정보들입니다. AWS 인증 정보와 같은 중요한 값들을 코드에 직접 노출시키지 않고 안전하게 사용할 수 있게 해줍니다. 이 프로젝트에서는 AWS 접근 키, 시크릿 키, 리전 정보 등을 저장하는데 사용됩니다.

## 배포 아키텍처

![Next js 배포 아키텍처 시퀀스 다이어그램](https://github.com/user-attachments/assets/2845201e-4b93-47b8-acc8-9d715fba04bf)

배포 아키텍처는 GitHub Actions를 통한 자동화된 빌드 및 배포 프로세스를 보여줍니다. 코드가 main 브랜치에 푸시되면 자동으로 빌드되어 S3에 업로드되고, CloudFront를 통해 전 세계 사용자에게 배포됩니다.

## 배포 파이프라인 가이드

이 프로젝트는 GitHub Actions를 사용해 AWS S3와 CloudFront를 기반으로 배포를 자동화(CI/CD)합니다. 아래는 배포 파이프라인에 대한 가이드입니다.

![AWS 배포 구조](https://github.com/user-attachments/assets/9f10c44c-6538-484a-aed9-b6cc4e053ad9)

### 배포 파이프라인의 주요 특징

#### CI/CD 구현

- **Continuous Integration (CI)**:  
  main 브랜치로 푸시되면 자동으로 빌드와 테스트가 실행됩니다.
- **Continuous Deployment (CD)**:  
  성공적으로 빌드된 결과물을 S3 버킷에 업로드하고 CloudFront 캐시를 무효화하여 최신 버전을 사용자에게 제공합니다.

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

### 전체 워크플로우 파일

다음은 전체 `deployment.yml` 파일의 내용입니다:

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

## CDN과 성능 최적화

### CDN(Content Delivery Network) 개요

CDN은 전 세계에 분산된 서버 네트워크를 통해 웹 콘텐츠를 빠르고 안정적으로 제공하는 시스템입니다.

- 웹사이트 로딩 속도 향상
- 서버 부하 감소
- 전반적인 사용자 경험 개선

### AWS CloudFront

AWS의 CDN 서비스인 CloudFront는 전 세계 엣지 로케이션을 통해 콘텐츠를 빠르게 전달합니다.

#### 엣지 로케이션 동작 방식

1. 사용자가 웹사이트 접속
2. DNS가 가장 가까운 CloudFront 엣지 로케이션으로 라우팅
3. 엣지 로케이션에서 요청 콘텐츠 확인
4. 캐시된 콘텐츠가 있으면 즉시 응답

### 성능 비교: CloudFront vs S3

#### CloudFront를 통한 콘텐츠 전달

![CloudFront](https://github.com/user-attachments/assets/7597f03d-daff-4979-9b05-db39b64d3a44)

- 콘텐츠 압축 제공 (`Content-Encoding` 헤더 존재)
- 캐시 적용 (`X-Cache: Hit from cloudfront` 헤더)
- 빠른 응답 속도
- 작은 파일 사이즈

#### S3 직접 접근

![s3](https://github.com/user-attachments/assets/688e9019-f965-48c2-895f-aa0eca582294)

- 콘텐츠 압축 없음 (`Content-Encoding` 헤더 없음)
- 상대적으로 느린 응답 속도
- 원본 크기의 파일 전송

### CloudFront 도입 후 성능 개선 결과

![성능 비교](https://github.com/user-attachments/assets/164652f3-2cc7-4dd4-bb08-18b7faa6fead)

#### CloudFront vs S3 성능 비교

| 파일 종류  | S3 용량 | CloudFront 용량 | 감소율 | S3 로딩 | CloudFront 로딩 | 개선율 |
| ---------- | ------- | --------------- | ------ | ------- | --------------- | ------ |
| HTML       | 12.6 KB | 3.3 KB          | 74%    | 826ms   | 595ms           | 28%    |
| CSS        | 9.1 KB  | 2.9 KB          | 68%    | 948ms   | 629ms           | 34%    |
| JavaScript | 166 KB  | 50.2 KB         | 70%    | 2.37s   | 1.12s           | 53%    |

### 전체 성능 개선 요약

- **평균 파일 크기 감소율**: 약 71%
- **평균 로딩 시간 개선율**: 약 38%
- **총 전송 크기**: 574KB → 292KB (약 49% 감소)
- **총 로딩 시간**: 5.89s → 4.40s (약 25% 단축)

CloudFront 도입으로 파일 크기는 평균 71% 감소했으며, 로딩 시간은 평균 38% 단축되어 웹사이트의 성능이 크게 개선되었습니다.

## 개발 환경 설정

### 요구사항

- Node.js 18.x 이상
- npm, yarn 또는 pnpm

### 프로젝트 설정

```bash
# 저장소 클론
git clone [repository-url]

# 의존성 설치
npm install
# or
yarn install
# or
pnpm install

# 개발 서버 실행
npm run dev
# or
yarn dev
# or
pnpm dev
```
