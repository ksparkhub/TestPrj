# pom.xml에서 변수를 참조하는 두 가지 방법

Maven 빌드 시 `pom.xml` 파일에서 외부 변수(예: GitHub Actions Secret)를 참조하는 두 가지 주요 방법과 그 차이점을 설명합니다.

## 1. OS 환경 변수 참조 (`${env.변수명}`)

이 방식은 운영 체제(OS)에 설정된 환경 변수를 직접 참조합니다.

### 설정 방법

**`.github/workflows/ci.yml`**

GitHub Actions 워크플로우의 `steps`에서 `env` 키를 사용하여 OS 환경 변수를 설정합니다.

```yaml
- name: Build with Maven
  env:
    # GitHub Secret을 OS 환경 변수로 설정
    DB_URL_DEV: ${{ secrets.DB_URL_DEV }}
    DB_USERNAME_DEV: ${{ secrets.DB_USERNAME_DEV }}
    DB_PASSWORD_DEV: ${{ secrets.DB_PASSWORD_DEV }}
  run: ./mvnw clean package -Dspring.profiles.active=dev
```

**`pom.xml`**

`properties` 태그 안에서 `${env.변수명}` 구문을 사용하여 해당 OS 환경 변수의 값을 읽어옵니다.

```xml
<properties>
    <db.url>${env.DB_URL_DEV}</db.url>
    <db.username>${env.DB_USERNAME_DEV}</db.username>
    <db.password>${env.DB_PASSWORD_DEV}</db.password>
</properties>
```

- **핵심**: `env.` 접두사는 Maven에게 "이것은 OS 레벨의 환경 변수이니 거기서 값을 찾아라"라고 알려주는 역할을 합니다.

## 2. JVM 시스템 속성 참조 (`${변수명}`)

이 방식은 Maven을 실행하는 JVM(자바 가상 머신)에 직접 '시스템 속성(System Property)'을 전달하여 참조합니다.

### 설정 방법

**`.github/workflows/ci.yml`**

Maven 실행 명령어(`run`)에 `-D` 플래그를 추가하여 JVM 시스템 속성을 직접 전달합니다. `env` 블록은 사용되지 않습니다.

```yaml
- name: Build with Maven
  run: >
    ./mvnw clean package -Dspring.profiles.active=dev
    -DDB_URL_DEV=${{ secrets.DB_URL_DEV }}
    -DDB_USERNAME_DEV=${{ secrets.DB_USERNAME_DEV }}
    -DDB_PASSWORD_DEV=${{ secrets.DB_PASSWORD_DEV }}
```

**`pom.xml`**

`properties` 태그 안에서 `${변수명}` 구문을 사용하여 해당 시스템 속성의 값을 읽어옵니다. `env.` 접두사가 없습니다.

```xml
<properties>
    <db.url>${DB_URL_DEV}</db.url>
    <db.username>${DB_USERNAME_DEV}</db.username>
    <db.password>${DB_PASSWORD_DEV}</db.password>
</properties>
```

- **핵심**: `-D` 플래그는 "이 키-값 쌍을 이번에 실행되는 JVM의 시스템 속성으로 사용해라"라는 의미입니다.

## 핵심 차이점 요약

| 구분 | OS 환경 변수 (`env.`) | JVM 시스템 속성 (`-D`) |
|---|---|---|
| **전달 방식** | `env:` 블록으로 쉘 환경에 설정 | `run` 명령어에 `-D` 플래그로 직접 전달 |
| **참조 범위** | 쉘 프로세스 전체에서 접근 가능 | 해당 Maven을 실행하는 JVM 프로세스 내에서만 유효 |
| **`pom.xml` 문법** | `${env.변수명}` | `${변수명}` |
| **특징** | OS 레벨의 일반적인 방식 | Java/Maven에 더 특화된 방식 |

두 방법 모두 외부의 값을 빌드 시점에 주입하는 동일한 목표를 달성하지만, 값이 전달되고 참조되는 기술적인 경로와 범위에서 차이가 있습니다.
