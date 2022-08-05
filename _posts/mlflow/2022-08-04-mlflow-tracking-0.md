---
title: "Tracking-0] tracking & tracking server"
excerpt: ""

categories:
  - MLflow
tags:
  - []

toc: true
toc_sticky: true

date: 2022-08-04T17:33:00+09:00
last_modified_at: 2022-08-04T17:33:00+09:00
---

# MLflow Tracking

MLflow Tracking 컴포넌트는 파라미터, 코드 버전, 매트릭, 파일 등을 기록하고 시각화는 API, UI를 포함한다.

## run & experiment

### run

코드의 실행을 run이라 표현한다. Tracking 컴포넌트는 run이 끝나며 다음 내용들을 기록한다.

* 코드 버전
  * 깃헙 커밋 해쉬
* 시작 & 종료 시간
* 소스
  * 프로젝트 이름, run을 시작하는 코드
* 파라미터
  * key-value 형태, string 타입으로 기록
* 메트릭
  * key-value 형태, numeric 타입으로 기록
  * run이 실행되는 동안 지속적으로 업데이트하며 변화를 기록할 수 있다.
* 아티팩트
  * 어떤 파일이던지 기록할 수 있다.
  * 주로 학습된 모델, 데이터, 이미지 등

파일 형태는 `artifacts`, 파라미터, 메트릭, 메타데이터 등은 `entity`라고 표현한다.

### experiment

run 그룹, 묶음. 테스크마다 experiment를 생성하여 experiment 아래에서 run을 실행한다.

## 엔티티, 아티팩트는 어디에 기록되는가?

다음 세 가지 저장소가 이용된다.

* local
* SQLAlchemy compatible database
  * SQLAlchemy: 파이썬 SQL 툴, 파이썬으로 고효율, 고성능 데이터베이스 엑세스를 가능하게 한다.
* remote tracking server

기본설정은 로컬에 저장하는 방식으로 run을 실행한 디렉토리에 `./mlruns` 디렉토리가 생성되고 하위에 experiment, run 별로 디렉토리가 생성되어 아티팩트, 엔티티가 기록된다.

```text
mlruns/
├── 0  # experiment ID
│   └── meta.yaml
├── 1
│   ├── 06ceabe1d4384f38b67fc35766723baa  # run hash
│   │   ├── artifacts
│   │   │   └── model
│   │   │       ├── MLmodel
│   │   │       ├── conda.yaml
│   │   │       ├── model.pkl
│   │   │       ├── python_env.yaml
│   │   │       └── requirements.txt
│   │   ├── meta.yaml
│   │   ├── metrics
│   │   │   ├── mae
│   │   │   ├── r2
│   │   │   └── rmse
│   │   ├── params
│   │   │   ├── alpha
│   │   │   └── l1_ratio
│   │   └── tags
│   │       ├── mlflow.log-model.history
│   │       ├── mlflow.project.backend
│   │       ├── mlflow.project.entryPoint
│   │       ├── mlflow.source.name
│   │       ├── mlflow.source.type
│   │       └── mlflow.user
...
```

`mlflow.set_tracking_uri()`를 호출하거나 환경변수 `MLFLOW_TRACKING_URI`를 통해 저장소를 설정할 수 있다.

* local - 엔티티, 아티팩트 기록
  * ex) `file:/my/local/dir`
  * local에 파일로 저장
* DB - 엔티티 기록
  * ex) `<dialect>+<driver>://<username>:<password>@<host>:<port>/<database>`
  * mysql, mssql, sqlite, postgresql
* HTTP 서버
  * ex) `https://my-server:5000`
* Databricks
  * 데이터 웨어하우스와 데이터 레이크를 결합한 아키텍쳐?


## 엔티티, 아티팩트는 어떻게 기록되는가?

두 가지 요소가 이용된다.
* 엔티티를 기록하는 `backend store`
* 아티팩트를 기록하는 `artifact store`

### 시나리오 1. localhost

![Scenario 1](https://mlflow.org/docs/latest/_images/scenario_1.png){:.align-center}

기본 설정. MLflow 클라이언트가 엔티티와 아티팩트를 직접 로컬에 기록한다. 로컬에 `./mlruns` 디렉토리가 생성된다.

```text
mlruns/
└── 0
    ├── 4b0793a1244f440f94c4edfff22fdd8e
    │   ├── artifacts
    │   │   └── model
    │   ├── meta.yaml
    │   ├── metrics
    │   │   ├── mae
    │   │   ├── r2
    │   │   └── rmse
    │   ├── params
    │   │   ├── alpha
    │   │   └── l1_ratio
    │   └── tags
    │       ├── mlflow.log-model.history
    │       ├── mlflow.project.backend
    │       ├── mlflow.project.entryPoint
    │       ├── mlflow.source.name
    │       ├── mlflow.source.type
    │       └── mlflow.user
    └── meta.yaml

```

### 시나리오 2. localhost + DB

![Scenario 2](https://mlflow.org/docs/latest/_images/scenario_2.png){:.align-center}

`export MLFLOW_TRACKING_URI=sqlite:///mlruns.db`로 트래킹 URI를 설정했는데 `mlruns` 디렉토리와 `mlruns.db` 파일이 생성되지만 run id를 찾을 수 없다는 예외가 발생하고 로깅이 안된다. URI를 다른 방식으로 설정해야하나...
{:.notice--danger}


### 시나리오 3. localhost + local Tracking Server

![Scenario 3](https://mlflow.org/docs/latest/_images/scenario_3.png){:.align-center}

로컬에 트래킹 서버를 띄운다. MLflow 클라이언트는 아티팩트는 직접 로컬 디렉토리에 기록하고 엔티티는 트래킹서버에 기록을 요청한다.

```bash
# 트래킹 서버 실행
mlflow server \
  --backend-store-uri sqlite:///mydb.sqlite \  # sqlite db에 엔티티 저장
  --default-artifact-root ./mlruns  # 로컬 디렉토리에 아티팩트 저장

# 트래킹 서버 URI 설정
export MLFLOW_TRACKING_URI="127.0.0.1:5000"

mlflow run ...
```

### 시나리오 4. remote tracking server, backend store, artifact store

![Scenario 4](https://mlflow.org/docs/latest/_images/scenario_4.png){:.align-center}

트래킹 서버, 백엔드, 아티팩트 저장소를 모두 원격 서버를 이용하는 방식. MLflow 클라이언트는 트래킹 서버로부터 아티팩트 저장소 URI를 얻어 아티팩트를 직접 전송하고 엔티티는 트래킹 서버를 통해 백엔드 저장소에 기록한다.

### 시나리오 5. remote tracking server enabled with proxied artifact storage access

![Scenario 5](https://mlflow.org/docs/latest/_images/scenario_5.png){:.align-center}

트래킹 서버가 아티팩트 관련 작업에 프록시 역할을 수행한다. MLflow 클라이언트가 직접 아티팩트 저장소에 접속하지않고 트래킹 서버를 통해 아티팩트를 기록한다. 아티팩트 저장소의 권한 관리 등에서 유리하다.

```bash
mlflow server \
  --backend-store-uri postgresql://URI \
  --artifacts-destination s3://bucket_name/mlartifacts \ # 아티팩트 저장소 URI
  --serve-artifacts \  # 트래킹 서버가 프록시 역할을 하여 아티팩트 관련 작업 대행
  --host ...
```

### 시나리오 6. remote tracking server used exclusively as proxied access host for artifact storage access

![Scenario 6](https://mlflow.org/docs/latest/_images/scenario_6.png){:.align-center}

시나리오 5의 형태에서 아티팩트 관련 작업 외의 동작은 수행하지 않는다. `--artifacts-only` 설정을 덧붙여 서버를 실행한다.



# 참고
* [MLflow](#https://mlflow.org/docs/latest/index.html){: .btn .btn--info}
* [MLflow Concepts](#https://mlflow.org/docs/latest/concepts.html){: .btn .btn--info}
* [MLflow Tutorials and Examples](#https://mlflow.org/docs/latest/tutorials-and-examples/index.html){: .btn .btn--info}