# JBoss Elytron Credential Store – CLI 구성 및 설명

이 문서는 JBoss EAP / WildFly 환경에서 **Elytron Credential Store**를 구성하기 위한 CLI 명령을 제공하고, 각 명령어의 **설명과 역할**을 함께 정리합니다.

---

## CLI 명령 + 상세 설명

### 1. Credential Store 생성

```bash
/subsystem=elytron/credential-store=cs:add(path=cs.ks, relative-to=jboss.server.config.dir, credential-reference={clear-text="StorePassword"}, create=true)
```

- **path=cs.ks**: 저장될 credential store 파일명
- **relative-to=jboss.server.config.dir**: 상대 경로 (기본은 `$JBOSS_HOME/standalone/configuration`)
- **credential-reference**: Store 자체를 여는 암호 (이게 없으면 cs.ks 못 엶)
- **create=true**: 파일이 없으면 새로 생성

---

### 2. alias 등록 (비밀번호 저장)

```bash
/subsystem=elytron/credential-store=cs:add-alias(alias=dbtest-password, secret-value=dbtest)
```

- **alias**: 나중에 datasource에서 참조할 이름
- **secret-value**: 실제 DB 접속 비밀번호 (입력 시 암호화되어 저장됨)

---

### 3. alias 목록 확인

```bash
/subsystem=elytron/credential-store=cs:read-aliases()
```

- 등록된 모든 alias 이름만 보여줌
- 비밀번호(값)는 절대 출력되지 않음

---

### 4. 기존 datasource에서 security-domain 제거

```bash
/subsystem=datasources/data-source=postgres:undefine-attribute(name=security-domain)
```

- 기존 PicketBox 방식의 security-domain 사용 중이면 제거 필요

---

### 5. datasource에 credential-reference 적용

```bash
/subsystem=datasources/data-source=postgres:write-attribute(name=credential-reference, value={store="cs", alias="dbtest-password"})
```

- 저장된 alias를 통해 비밀번호를 참조하도록 설정
- `store="cs"`는 cs.ks 파일, `alias`는 앞서 등록한 키

---

## XML 구성 예시

```xml
<security>
  <user-name>dbtest</user-name>
  <credential-reference store="cs" alias="dbtest-password"/>
</security>
```

---

## 적용 마무리

```bash
:reload
```

---

## 운영 팁

- alias는 내부에서만 복호화됨 → 평문 확인은 절대 불가
- cs.ks 또는 StorePassword 유실 시 alias 무용지물
- 반드시 CLI 등록 명령어 또는 평문 비밀번호 따로 백업

---

## 백업 CLI 예시

```bash
/subsystem=elytron/credential-store=cs:add-alias(alias=dbtest-password, secret-value=dbtest)
/subsystem=elytron/credential-store=cs:add-alias(alias=backup-user, secret-value=securePwd2025)
``
