# K8S

## Container Orchestration


### K8S

- 스케쥴링
- 자원 할당
- Service Discorvery
- 스케일링
- 로드밸런싱
- Self Healing
- Rollback / Rollout
- 설정 관리
- 스토리지 오케스트레이션


여러 머신으로 구성된 클러스터 상에서 컨테이너를 효율적으로 관리하기 위한 시스템


### Prerequisite

- EKS는 VPC 안에서 돌아간다. 따라서 테스트용 VPC 환경을 구축해야한다.
- AWS에서 제공하는 샘플 VPC cloudformation 양식과 비슷하게 `serverless.yml` 로 작성하였다.

배포는 `sls deploy --stage dev --region ap-northeast-2 --profile your-profile` 로 배포한다.

만약 `default` profile로 배포하고싶으면 `--profile` 생략해서 배포해도 무방하며, `stage` 또한 생략해도 무방하다.

### 샘플 VPC 도식

- 작성 예정
