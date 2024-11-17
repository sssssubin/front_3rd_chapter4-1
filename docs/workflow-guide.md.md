# GitHub Actions 워크플로우 설정 가이드

## 워크플로우 파일 생성
프로젝트 루트의 `.github/workflows/deployment.yml` 파일에 다음 내용을 작성합니다.

## 워크플로우 구성 요소 설명

### 1. 워크플로우 이름 설정
```yaml
name: Deploy Next.js to S3 and invalidate CloudFront
```
- GitHub Actions 대시보드에 표시될 워크플로우의 이름을 정의합니다.

### 2. 트리거 조건 설정
```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```
- `push`: main 브랜치에 코드가 푸시될 때 자동으로 실행됩니다.
- `workflow_dispatch`: GitHub UI에서 수동으로 워크플로우를 실행할 수 있습니다.

### 3. 작업 환경 설정
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```
- `jobs`: 실행할 작업들을 정의합니다.
- `deploy`: 작업의 이름입니다.
- `runs-on`: 최신 우분투 환경에서 작업이 실행됩니다.

### 4. 실행 단계 설정
#### 소스코드 체크아웃
```yaml
steps:
  - name: Checkout repository
    uses: actions/checkout@v2
```
- 소스 코드를 가져오는 첫 단계입니다.

#### Node.js 설정
```yaml
- name: Setup Node.js
  uses: actions/setup-node@v2
  with:
    node-version: "18.x"
```
- Node.js 18.x 버전을 설정합니다.

#### 의존성 설치
```yaml
- name: Install dependencies
  run: npm ci
```
- `npm ci` 명령어로 프로젝트 의존성을 설치합니다.
- `npm install`보다 더 엄격하고 일관된 설치를 보장합니다.

#### 프로젝트 빌드
```yaml
- name: Build
  run: npm run build
```
- Next.js 프로젝트를 빌드합니다.
- 결과물은 `out` 디렉토리에 생성됩니다.

#### AWS 인증 설정
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

#### S3 배포
```yaml
- name: Deploy to S3
  run: |
    aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete
```
- 빌드된 파일들을 S3 버킷에 동기화합니다.
- `--delete` 옵션은 S3에는 있지만 로컬에는 없는 파일들을 삭제합니다.

#### CloudFront 캐시 무효화
```yaml
- name: Invalidate CloudFront cache
  run: |
    aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```
- CloudFront 캐시를 무효화합니다.
- `/*`는 모든 경로의 캐시를 무효화한다는 의미입니다.
- 사용자들이 항상 최신 버전의 웹사이트를 볼 수 있도록 합니다.