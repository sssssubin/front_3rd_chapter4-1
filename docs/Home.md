# Next.js 자동 배포 CI/CD 파이프라인 Wiki

## 목차

1. [워크플로우 설정 가이드](workflow-guide.md)
2. [CDN과 성능 최적화](cdn-performance.md)

## 주요 링크 및 개념

### 주요 링크

- S3 버킷 웹사이트 엔드포인트: [http://subin-s3-test.s3-website-ap-southeast-2.amazonaws.com/](http://subin-s3-test.s3-website-ap-southeast-2.amazonaws.com/)
- CloudFrount 배포 도메인 이름: [https://d2icfx053w3v0p.cloudfront.net/](https://d2icfx053w3v0p.cloudfront.net/)

### 주요 개념

#### GitHub Actions과 CI/CD 도구

- GitHub에서 제공하는 자동화된 워크플로우 도구
- 코드 변경사항 발생 시 자동으로 빌드, 테스트, 배포 수행
- 지속적 통합(CI)과 지속적 배포(CD) 가능

#### S3와 스토리지

- Amazon S3(Simple Storage Service)는 AWS의 클라우드 스토리지 서비스
- 정적 웹사이트 호스팅 가능
- 높은 내구성과 가용성 제공
- 빌드된 Next.js 애플리케이션의 정적 파일 저장 및 서비스

#### CloudFront와 CDN

- Amazon CloudFront는 AWS의 콘텐츠 전송 네트워크(CDN) 서비스
- 전 세계 엣지 로케이션을 통한 콘텐츠 캐싱
- S3 저장 파일의 전 세계 빠른 전송

#### 캐시 무효화(Cache Invalidation)

- CloudFront 엣지 로케이션의 캐시 강제 새로고침
- 새 버전 배포 시 실행
- 최신 버전 콘텐츠 제공 보장

#### Repository secret과 환경변수

- GitHub 저장소의 안전한 민감 정보 보관
- AWS 인증 정보 등 중요 값 보호
- AWS 접근 키, 시크릿 키, 리전 정보 등 저장
