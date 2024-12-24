# 프로젝트 제목

이 문서는 한글 버전입니다. 영어 버전을 보려면 [여기](README.md)를 클릭하세요.
# OpenMSA

<img src="https://github.com/user-attachments/assets/a1ffb8f0-62b0-42be-ad33-2f2aecbe5116" alt="예시 이미지" width="500">

OpenMSA는 다중 마스터 구성과 로드 밸런싱을 지원하는 올인원 클러스터 관리 솔루션으로, 여러 Kubernetes 배포판(RKE2, Kubeadm, K3S)을 지원하며 카탈로그 서비스 배포를 자동화합니다.

## 목차
- [시스템 아키텍처](#시스템-아키텍처)
- [시스템 요구사항](#시스템-요구사항)
- [지원 인프라](#지원-인프라)
- [설치 과정](#설치-과정)
- [설치 후 작업](#설치-후-작업)
- [구성 세부사항](#구성-세부사항)

## 시스템 아키텍처

### 핵심 구성요소
- **Go 바이너리**: OpenMSA 주 실행 파일
- **GitHub Releases**: 바이너리 및 설치 스크립트 배포 플랫폼
- **설치 스크립트**: 시스템 종속성 자동 설정
- **Ansible 스크립트**: OS 및 Kubernetes 변형에 따른 설치 오케스트레이션
- **구성 디렉토리**: `/etc/openmsa`에 수정 가능한 Ansible 플레이북 포함

## 시스템 요구사항

- 최소 3개의 노드
- 노드별 요구사항:
  - 메모리: 8GB 이상
  - CPU: 4코어 이상
  - 시스템 및 컨테이너 저장소를 위한 충분한 디스크 공간

## 지원 인프라

### 검증된 운영체제
- AWS Amazon Linux 2
- Ubuntu 22.04 LTS
- Rocky Linux 8.9

### 노드 구성 규칙

| 노드 유형 | 역할 | 레이블 | 설명 |
|----------|------|--------|------|
| MGMT 노드 | worker | control=true | 로드밸런서와 Ansible Tower를 위한 MGMT(제어) 노드 |
| 마스터 노드 | control | master=true | 컨트롤 플레인 노드 |
| 워커 노드 | control-plane | worker=true | 워크로드 실행을 위한 워커 노드 |

### 지원되는 카탈로그 서비스

| 카탈로그 이름 | 기본값 | 노드-셀렉터 | 설명 |
|--------------|--------|--------------|------|
| ArgoCD | False | - | CI/CD 자동화 서버 |
| Jenkins | False | - | CI/CD 자동화 서버 |
| Gitlab | False | - | 소스 코드 관리 |
| Keycloak | False | - | 신원 관리 |
| Prometheus Stack | True | - | 모니터링 솔루션 |
| Opensearch | False | - | 검색 및 분석 |
| Opensearch Dashboard | False | - | 시각화 플랫폼 |
| Fluent-bit | False | - | 로그 프로세서 |
| Rancher Dashboard | True | - | Kubernetes 관리 UI |
| Jaeger | False | - | 분산 추적 |
| MySQL | False | db=true | 관계형 데이터베이스 |
| MariaDB | False | db=true | 관계형 데이터베이스 |
| PostgreSQL-HA | False | db=true | HA PostgreSQL 클러스터 |
| Redis-cluster | False | db=true | 인메모리 데이터 저장소 |
| Kafka | False | - | 이벤트 스트리밍 플랫폼 |
| Longhorn | True | storage=true | 분산 스토리지 |
| Minio | False | - | S3 호환 스토리지 |
| Istio | False | - | 서비스 메시 |
| Velero | False | - | 백업 및 마이그레이션 |
| Neuvector | False | - | 컨테이너 보안 |

### Kubernetes 변형
- RKE2
- Kubeadm
- K3S

### 로드 밸런싱
- 분산 로드 관리를 위한 HAProxy 구현

## 설치 과정

### 전제 조건
- 비밀번호 인증이 가능한 root 계정 접근
- 시스템 수정에는 root 권한 필요
- 패키지 다운로드를 위한 인터넷 연결

### 빠른 시작

1. 설치 스크립트 다운로드 및 실행:
```bash
curl -L https://github.com/yuseok-jeong/openmsa-releases/releases/download/installer-v1.0.0/install.sh | sudo bash
```

2. OpenMSA 실행:
```bash
openmsa
```

### 설치 단계

설치 과정은 다음을 포함합니다:
1. OpenMSA 바이너리 배포
2. Ansible 구성
3. OpenSSH 설정 (Ansible 자동화에 필요)
4. Python 설치 (Ansible 종속성)

### 시스템 구성
- 서버 정보 구성
- Ansible 연결 설정
- Kubernetes 배포 유형 선택
- 원하는 카탈로그 옵션 선택

## 설치 후 작업

### 클러스터 확인

Kubernetes 클러스터 상태 확인:
```bash
kubectl get nodes
kubectl get pods -A
```

클러스터 관리를 위한 K9s 사용:
```bash
k9s
```

### 카탈로그 서비스 접근

1. hosts 파일 업데이트:
   - `/etc/hosts`를 편집하여 MGMT 노드의 IP 주소 포함
2. 구성된 인그레스 주소를 통해 카탈로그 UI 접근
3. 각 배포된 서비스가 정상 작동하는지 확인

## 구성 세부사항

### 구성 관리
- 모든 Ansible 플레이북은 `/etc/openmsa` 디렉토리에 위치
- 특정 요구사항에 따라 플레이북 커스터마이징 가능

### 자동화된 작업
시스템은 다음을 자동으로 관리합니다:
- 운영체제별 구성
- Kubernetes 변형 배포
- 카탈로그 서비스 설치 및 구성

## 라이선스

이 프로젝트는 [LICENSE.md](LICENSE.md) 파일에 명시된 라이선스를 따릅니다.

## 지원

지원이 필요한 경우 GitHub 저장소에 이슈를 생성해 주세요.
