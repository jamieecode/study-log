# docker

# docker compose build  시 `.env.development`를 읽지 못해 실패한 이슈

## 이슈 개요

Next.js 16.0.10 버전 프로젝트에서 개발 환경 변수는 `.env.development` 파일로 관리하고 있었다.

VSCode 환경에서 `docker compose build` 실행 시 `.env.development` 파일을 읽지 못해 빌드가 실패하는 문제가 발생했다.

임시로 `.env` 파일을 생성하여 동일한 환경 변수를 작성한 뒤 빌드를 수행하자 정상적으로 성공했다.

원인을 분석한 결과, Docker Compose의 기본 환경 변수 로드 규칙에 따른 동작임을 확인했다.

---

## 1. Docker Compose의 기본 환경 변수 로드 규칙

Docker Compose는 별도의 설정이 없는 경우, **현재 디렉토리의 `.env` 파일만 자동으로 로드**한다.

- **`.env`**
    - Docker Compose에서 예약된 기본 파일명이다.
    - `docker-compose.yml` 내에서 `${VAR}` 형태의 변수 치환(interpolation)을 위해 자동으로 로드된다.
- **`.env.development`**
    - Docker Compose 기준에서는 단순한 텍스트 파일이다.
    - 자동으로 환경 변수로 인식하거나 로드되지 않는다.

이로 인해 `.env.development`에만 환경 변수가 정의된 경우, Docker Compose는 해당 변수를 알 수 없게 된다.

---

## 2. 빌드 시점(Build-time)과 실행 시점(Runtime)의 차이

Next.js 16과 같은 프레임워크는 **빌드 시점에 환경 변수가 필요한 구조**를 가진다.

- **변수 치환 문제**
    - `docker-compose.yml`에서 `${VAR}` 형태를 사용할 경우, Docker Compose는 `.env` 파일에서 값을 찾는다.
    - `.env.development`에만 변수가 존재하면 빌드 단계에서 값을 찾지 못해 빈 값으로 처리된다.
- **Next.js 특성**
    - `NEXT_PUBLIC_` 접두사가 붙은 환경 변수는 빌드 시점에 브라우저 코드에 포함된다.
    - Docker Compose가 `.env.development`를 무시한 상태로 빌드를 수행하면, 해당 변수들은 `undefined`가 되어 애플리케이션이 정상적으로 동작하지 않는다.

---

## 3. `.env.development`를 사용하는 방법

기본 파일인 `.env` 대신 다른 환경 변수 파일을 사용하려면 **명시적인 설정이 필요**하다.

### 1) CLI 옵션 사용

- `-env-file` 옵션을 사용해 Docker Compose에 사용할 환경 변수 파일을 직접 지정할 수 있다.

```bash
docker compose --env-file .env.development build
```

이 방식은 빌드 단계에서 변수 치환까지 정상적으로 적용된다.

---

### 2) `docker-compose.yml`의 `env_file` 사용 시 주의점

`env_file` 옵션은 **컨테이너 실행 시점(Runtime)** 에 환경 변수를 전달하는 용도이다.

- 컨테이너 내부에는 환경 변수가 전달된다.
- 하지만 `docker-compose.yml` 자체의 변수 치환이나 빌드 인수(ARG)에는 영향을 주지 않을 수 있다.

따라서 **빌드 시점에 환경 변수가 필요한 경우**에는 `--env-file` 옵션 또는 `.env` 파일을 사용하는 방식이 더 안전하다.

---

---

# Issue: Docker Compose Build Failure Due to `.env.development` Not Being Loaded

## Overview

In a Next.js 16.0.10 project, development environment variables were managed using a `.env.development` file.

When running `docker compose build` in a VSCode environment, the build failed because Docker Compose could not read the `.env.development` file.

As a temporary workaround, a `.env` file containing the same environment variables was added, and the build succeeded.

Further investigation revealed that this behavior was caused by Docker Compose’s default environment variable loading rules.

---

## 1. Default Environment Variable Loading Rules in Docker Compose

By default, Docker Compose **automatically loads only the `.env` file** located in the current directory.

- **`.env`**
    - A reserved default filename for Docker Compose.
    - Automatically loaded for variable interpolation (`${VAR}`) in `docker-compose.yml`.
- **`.env.development`**
    - Treated as a plain text file by Docker Compose.
    - Not automatically recognized or loaded as environment variables.

As a result, variables defined only in `.env.development` are ignored during the build process.

---

## 2. Build-time vs Runtime Environment Variables

Frameworks like Next.js 16 require environment variables **at build time**, not just at runtime.

- **Variable interpolation issue**
    - When `${VAR}` is used in `docker-compose.yml`, Docker Compose looks for the value in the `.env` file.
    - If the variable exists only in `.env.development`, it is treated as an empty value during the build.
- **Next.js behavior**
    - Environment variables prefixed with `NEXT_PUBLIC_` are embedded into the client-side code at build time.
    - If Docker Compose ignores `.env.development`, these variables become `undefined`, causing the application to malfunction.

---

## 3. How to Use `.env.development`

To use an environment variable file other than `.env`, explicit configuration is required.

### 1) Specify the file via CLI

The `--env-file` option allows you to explicitly define which environment variable file Docker Compose should use.

```bash
docker compose --env-file .env.development build
```

This ensures that variable interpolation works correctly during the build phase.

---

### 2) Caution When Using `env_file` in `docker-compose.yml`

The `env_file` option is intended to pass environment variables **into the running container (runtime)**.

- Environment variables are available inside the container.
- However, they may not be applied to variable interpolation or build arguments in `docker-compose.yml`.

Therefore, if environment variables are required at **build time**, using `--env-file` or a `.env` file is the safer approach.