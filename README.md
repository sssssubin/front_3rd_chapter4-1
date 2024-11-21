# Next.js CI/CD 파이프라인 구축 🚩

![CI/CD Status](https://github.com/sssssubin/front_3rd_chapter4-1/actions/workflows/deployment.yml/badge.svg)

> Last Updated: 2024.11.21

## 목차

1. [소개](#소개)
2. [특징](#특징)
3. [배포 시퀀스 다이어그램](#배포-시퀀스-다이어그램)
4. [배포 파이프라인](#배포-파이프라인)
5. [배포 설정](#배포-설정)
6. [시작하기](#시작하기)
7. [문서](#문서)

## 소개

이 프로젝트는 AWS S3와 CloudFront를 활용하여 Next.js 애플리케이션의 자동 배포를 위한 CI/CD 파이프라인을 구축합니다.

## 특징

- GitHub Actions를 통한 자동화된 빌드/배포
- AWS S3와 CloudFront를 활용한 효율적인 정적 호스팅
- 글로벌 CDN을 통한 성능 최적화 (평균 로딩시간 38% 개선)

## 배포 시퀀스 다이어그램

GitHub Actions를 통한 자동화된 빌드 및 배포 프로세스를 보여주는 시퀀스 다이어그램입니다. 코드가 main 브랜치에 푸시되면 자동으로 빌드되어 S3에 업로드되고, CloudFront를 통해 전 세계 사용자에게 배포됩니다.

![Next js 배포 아키텍처 시퀀스 다이어그램](https://github.com/user-attachments/assets/00586fb4-944b-42ec-be54-7bb7ee6779b9)

### 데이터 흐름

1. **저장소 (GitHub Repository)**
   - 소스 코드 저장 및 CI/CD 파이프라인 시작점
2. **스토리지 (S3 Bucket)**
   - 빌드된 정적 파일 저장
3. **CDN (CloudFront)**
   - 전 세계 사용자에게 최적화된 콘텐츠 전달

## 배포 파이프라인

이 프로젝트는 GitHub Actions를 사용해 AWS S3와 CloudFront를 기반으로 배포를 자동화(CI/CD)합니다.

![AWS 배포 구조](https://github.com/user-attachments/assets/9f10c44c-6538-484a-aed9-b6cc4e053ad9)

### CI/CD

- Continuous Integration (CI)
  - main 브랜치 푸시 시 자동 빌드 및 테스트 실행
- Continuous Deployment (CD)
  - 빌드 결과물 S3 버킷 업로드
  - CloudFront 캐시 무효화로 최신 버전 제공

## 배포 설정

### GitHub Secrets 설정

워크플로우 실행을 위해 GitHub 저장소의 Settings > Secrets and variables > Actions 메뉴에서 다음 값들을 설정해야 합니다:

- `AWS_ACCESS_KEY_ID`: AWS IAM 사용자의 액세스 키 ID
- `AWS_SECRET_ACCESS_KEY`: AWS IAM 사용자의 시크릿 액세스 키
- `AWS_REGION`: AWS 리전 (예: ap-northeast-2)
- `S3_BUCKET_NAME`: 정적 웹사이트 호스팅용 S3 버킷 이름
- `CLOUDFRONT_DISTRIBUTION_ID`: CloudFront 배포 ID

## 시작하기

### 요구사항

- Node.js 18.x 이상
- npm, yarn 또는 pnpm

### 설치 및 실행

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

## 문서

자세한 내용은 Wiki를 참고하세요:

- [GitHub Actions 워크플로우 설정 가이드](https://github.com/sssssubin/front_3rd_chapter4-1/wiki/workflow-guide.md)
- [CDN과 성능 최적화](https://github.com/sssssubin/front_3rd_chapter4-1/wiki/cdn-performance.md)
