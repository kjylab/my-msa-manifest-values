# my-msa-manifest-values

각 마이크로서비스의 **Helm values 파일** 모음. [my-market-msa-manifest](https://github.com/kjylab/my-market-msa-manifest) 공통 차트와 함께 사용된다.

## 개념

### 왜 values를 별도 레포로 분리하나?
- **보안**: DB 패스워드, 설정값 등을 서비스 소스코드 레포와 분리
- **배포 자동화**: CI(GitHub Actions)가 새 이미지 태그만 이 레포에 커밋 → ArgoCD가 감지해서 자동 배포
- **이력 관리**: 어느 시점에 어떤 이미지가 배포됐는지 Git 이력으로 추적 가능

### GitOps 배포 흐름

```
1. 소스코드 변경 → Git push (my-msa-product 레포)
2. GitHub Actions 실행
   a. JAR 빌드
   b. Docker 이미지 빌드 + Docker Hub push
   c. 이 레포의 values-release.yaml에서 tag를 커밋 SHA로 업데이트
3. ArgoCD가 이 레포 변경 감지
4. 새 이미지로 파드 롤링 업데이트
```

## 디렉토리 구조

```
product-service/
  values-release.yaml     # product-service Helm values

inventory-service/
  values-release.yaml     # inventory-service Helm values

order-service/
  values-release.yaml

user-api-gateway/
  values-release.yaml
```

## values-release.yaml 핵심 항목

```yaml
image:
  repository: jyupk/my-msa-product-service
  tag: 188f737412bc7f511c0208f02f46c24783fc229b  # ← CI가 자동 업데이트

serviceMonitor:
  enabled: true           # Prometheus 메트릭 수집 활성화

appConfig:
  spring:
    datasource:
      url: jdbc:postgresql://postgresql.postgresql.svc.cluster.local:5432/product_db
  management:
    endpoints:
      web:
        exposure:
          include: health,prometheus   # actuator 엔드포인트 노출
```

## 수동으로 이미지 롤백하는 방법

```bash
# values-release.yaml의 tag를 이전 커밋 SHA로 변경 후 push
# ArgoCD가 자동으로 이전 이미지로 롤백
git log --oneline  # 이전 커밋 SHA 확인
```
