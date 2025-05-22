# 🔐 JBoss Elytron Credential Store – 복호화 개념 및 한 줄 CLI 구성

이 문서는 JBoss EAP 7.x / WildFly에서 **Elytron Credential Store**를 활용하여  
비밀번호를 평문으로 남기지 않고, **런타임 시 내부적으로 복호화하여 사용하는 구조**를 설명하며,  
운영자가 `jboss-cli.sh`에서 쉽게 사용할 수 있도록 **모든 명령을 한 줄로 정리**합니다.

---

## ✅ 주요 CLI 명령 (한 줄 버전)

```bash
/subsystem=elytron/credential-store=cs:add(path=cs.ks, relative-to=jboss.server.config.dir, credential-reference={clear-text="StorePassword"}, create=true)

/subsystem=elytron/credential-store=cs:add-alias(alias=dbtest-password, secret-value=dbtest)

/subsystem=elytron/credential-store=cs:read-aliases()

/subsystem=datasources/data-source=postgres:undefine-attribute(name=security-domain)

/subsystem=datasources/data-source=postgres:write-attribute(name=credential-reference, value={store="cs", alias="dbtest-password"})
```

---

## 📄 datasource XML 설정 예시

```xml
<security>
  <user-name>dbtest</user-name>
  <credential-reference store="cs" alias="dbtest-password"/>
</security>
```

---

## 📁 KeyStore 파일 위치 확인

```bash
ls -l $JBOSS_HOME/standalone/configuration/cs.ks
```

---

## 🧾 구성 완료 후 반영

```bash
:reload
```

---

## ❌ 복호화는 가능한가?

| 항목 | 가능 여부 | 설명 |
|------|------------|------|
| alias 목록 확인 | ✅ | `read-aliases()` |
| alias 값 확인 (복호화) | ❌ | 평문은 절대 확인 불가 |
| 외부 도구(keytool 등)로 복호화 | ❌ | JCEKS 내부 구조 특성상 불가능 |

---

## 🛡️ 운영자 필수 주의 사항

- `secret-value`는 반드시 등록 시점에 알고 있어야 함
- alias만 저장되어 있어도 평문 모르면 복구 불가
- `cs.ks` 분실 시 모든 alias 내용도 유실됨
- 반드시 평문 또는 CLI 등록 스크립트 별도 백업 필요

---

## ✍️ 추천 백업 CLI 템플릿

```bash
/subsystem=elytron/credential-store=cs:add-alias(alias=dbtest-password, secret-value=dbtest)
/subsystem=elytron/credential-store=cs:add-alias(alias=backup-user, secret-value=securePwd2025)
```
