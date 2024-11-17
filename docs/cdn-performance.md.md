# CDN과 성능 최적화

## CDN(Content Delivery Network) 개요
CDN은 전 세계에 분산된 서버 네트워크를 통해 웹 콘텐츠를 빠르고 안정적으로 제공하는 시스템입니다.

### 주요 특징
- 웹사이트 로딩 속도 향상
- 서버 부하 감소
- 전반적인 사용자 경험 개선

## AWS CloudFront
- AWS의 CDN 서비스인 CloudFront는 전 세계 엣지 로케이션을 통해 콘텐츠를 빠르게 전달합니다.

### 엣지 로케이션 동작 방식
1. 사용자가 웹사이트 접속
2. DNS가 가장 가까운 CloudFront 엣지 로케이션으로 라우팅
3. 엣지 로케이션에서 요청 콘텐츠 확인
4. 캐시된 콘텐츠가 있으면 즉시 응답

## 성능 비교: CloudFront vs S3

### CloudFront를 통한 콘텐츠 전달
![CloudFront](https://github.com/user-attachments/assets/7597f03d-daff-4979-9b05-db39b64d3a44)

- 콘텐츠 압축 제공 (`Content-Encoding` 헤더 존재)
- 캐시 적용 (`X-Cache: Hit from cloudfront` 헤더)
- 빠른 응답 속도
- 작은 파일 사이즈

### S3 직접 접근
![s3](https://github.com/user-attachments/assets/688e9019-f965-48c2-895f-aa0eca582294)

- 콘텐츠 압축 없음 (`Content-Encoding` 헤더 없음)
- 상대적으로 느린 응답 속도
- 원본 크기의 파일 전송

## CloudFront 도입 후 성능 개선 결과
![성능 비교](https://github.com/user-attachments/assets/164652f3-2cc7-4dd4-bb08-18b7faa6fead)

### CloudFront vs S3 성능 비교

| 파일 종류  | S3 용량 | CloudFront 용량 | 감소율 | S3 로딩 | CloudFront 로딩 | 개선율 |
|------------|---------|-----------------|--------|----------|-----------------|---------|
| HTML       | 12.6 KB | 3.3 KB         | 74%    | 826ms    | 595ms          | 28%    |
| CSS        | 9.1 KB  | 2.9 KB         | 68%    | 948ms    | 629ms          | 34%    |
| JavaScript | 166 KB  | 50.2 KB        | 70%    | 2.37s    | 1.12s          | 53%    |

### 전체 성능 개선 요약
- **평균 파일 크기 감소율**: 약 71%
- **평균 로딩 시간 개선율**: 약 38%
- **총 전송 크기**: 574KB → 292KB (약 49% 감소)
- **총 로딩 시간**: 5.89s → 4.40s (약 25% 단축)

CloudFront 도입으로 파일 크기는 평균 71% 감소했으며, 로딩 시간은 평균 38% 단축되어 웹사이트의 성능이 크게 개선되었습니다.