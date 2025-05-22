# 🔐 JBoss Elytron Credential Store – 복호화 개념 및 적용 방법

이 문서는 JBoss EAP 7.x / WildFly에서 **Elytron Credential Store**를 활용하여  
비밀번호를 직접 평문으로 남기지 않고, **런타임 시 내부적으로 복호화하여 사용하는 구조**를 설명합니다.

---

## ✅ 개요

Credential Store는 민감 정보를 평문으로 직접 노출하지 않기 위해 사용하는 JBoss Elytron의 기능입니다.  
비밀번호는 `cs.ks`라는 KeyStore 파일에 암호화된 형태로 저장되며, JBoss 서버는 이를 **런타임에 복호화하여 사용**합니다.

---

## 🧩 주요 개념

|           항    목         |                   설         명                    | 
|----------------------------|----------------------------------------------------|
| Credential Store (`cs.ks`) | JCEKS 형식의 KeyStore 파일, 암호화된 비밀번호 저장 |
| Store Password             | cs.ks를 열기 위한 마스터 비밀번호                  |
| alias                      | 각각의 저장된 비밀번호를 식별하기 위한 이름        |
| secret-value               | alias에 등록되는 실제 평문 비밀번호                |
| credential-reference       | datasource 등에서 alias를 사용하는 방식            |

---

## 1. Credential Store 생성

```
/subsystem=elytron/credential-store=cs:add(
  path=cs.ks,
  relative-to=jboss.server.config.dir,
  credential-reference={clear-text="StorePassword"},
  create=true
)
```

---

## 2. 비밀번호 등록 (alias 생성)

```
/subsystem=elytron/credential-store=cs:add-alias(
  alias=dbtest-password,
  secret-value=dbtest
)
```

---

## 3. alias 목록 확인

```
/subsystem=elytron/credential-store=cs:read-aliases()
```

결과 예시:
```
{
  "outcome" => "success",
  "result" => ["dbtest-password"]
}
```

>  `secret-value`의 평문은 볼 수 없습니다.

---

## 4. datasource에 credential-reference 적용

기존 `<password>` 제거 후, 다음과 같이 설정:

```xml
<security>
  <user-name>dbtest</user-name>
  <credential-reference store="cs" alias="dbtest-password"/>
</security>
```

또는 CLI 적용:

```
/subsystem=datasources/data-source=postgres:undefine-attribute(name=security-domain)
/subsystem=datasources/data-source=postgres:write-attribute(
  name=credential-reference,
  value={store="cs", alias="dbtest-password"}
)
```

---

## 5. 반영 적용

```
:reload
```

---

## 6. KeyStore 파일 위치

```
ls -l $JBOSS_HOME/standalone/configuration/cs.ks
```

---

##  복호화는 가능한가?

|       항      목      | 가능 여부 |                       설명                     |
|-----------------------|-----------|------------------------------------------------|
| alias 목록 확인       | ✅       | `read-aliases()`                                |
| alias 값 확인 (복호화)| ❌       |   평문은 절대 확인 불가                         |

---

## 주의 사항

- 평문 비밀번호는 반드시 **등록 시점에만 알고 있어야** 하며,
- 추후 복구하려면 alias 재등록이 필요합니다
- `cs.ks` 분실 시, 복호화 불가 → 평문 다시 입력해야 함

---

##  권장 운영 정책

| 항목 | 권장 방법 |
|------|-----------|
| 평문 비밀번호 보관 | 안전한 위치에 따로 백업 (예: Ansible Vault, 비밀 저장소) |
| alias CLI 백업 | `register-aliases.cli` 파일로 관리 |
| cs.ks 백업 | config 디렉토리 내 KeyStore 파일 주기적 백업 |

---

##  요약

- Credential Store는 평문 비밀번호를 **직접 남기지 않기 위해 사전에 암호화하여 alias로 저장**합니다
- JBoss는 이 alias를 통해 런타임에 복호화하여 사용하지만, **운영자는 복호화 결과를 확인할 수 없습니다**
- 평문을 반드시 알고 있어야 alias를 등록할 수 있으며, 유실 시 복구 불가입니다

---

## CLI 백업 템플릿

```
/subsystem=elytron/credential-store=cs:add-alias(alias=dbtest-password, secret-value=dbtest)
/subsystem=elytron/credential-store=cs:add-alias(alias=backup-user, secret-value=securePwd2025)
```
