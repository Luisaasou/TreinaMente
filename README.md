# Projeto Segurança – Login, Auditoria, LGPD e API Externa

Projeto **Spring Boot 3 / Java 17 / Maven** que cobre os 4 itens da etapa:

| # | Requisito da etapa | Onde está no código |
|---|---|---|
| 1 | Login: **autenticação**, **autorização**, **criptografia** | `security/*`, `config/SecurityConfig`, `service/AuthService`, `controller/AuthController` |
| 2 | **Auditoria/Log** de acessos e ações | `audit/AuditInterceptor`, `service/AuditService`, `model/AuditLog`, `controller/AuditController` |
| 3 | **LGPD**: termo de uso e política de privacidade | `static/termos-de-uso.html`, `static/politica-de-privacidade.html`, `controller/LgpdController`, `controller/UserController` |
| 4 | **API externa** (projeto lógico + documentação) | `service/ViaCepService`, `controller/ExternalApiController`, `docs/INTEGRACAO_API_EXTERNA.md` |

## Como abrir no IntelliJ
1. Extraia o `.zip`.
2. **File > Open** e selecione o arquivo **`pom.xml`** (ou a pasta) > **Open as Project**.
3. **File > Project Structure > Project**: SDK = **JDK 17 ou superior**.
4. Aguarde o Maven baixar as dependências (barra de progresso no canto inferior).
5. Abra `SegurancaApplication.java` e clique no ▶ ao lado do `main`.
6. Acesse http://localhost:8080

## Como cada item funciona
**1) Login**
- *Autenticação:* `POST /api/auth/login` valida e-mail/senha e devolve um **token JWT** (validade de 1h).
- *Autorização:* o token vai no header `Authorization: Bearer <token>`. Perfis `USER` e `ADMIN`; `/api/audit/**` só ADMIN (ver `SecurityConfig`).
- *Criptografia:* senha salva com **BCrypt** (hash + salt); token assinado com HMAC-SHA.

**2) Auditoria** – Cada chamada em `/api/**` é gravada na tabela `audit_logs` (usuário, ação, rota, IP, status). Também são registrados: `REGISTRO_USUARIO`, `LOGIN_SUCESSO`, `LOGIN_FALHA`, `ACESSO_NEGADO`, `ACESSO_NAO_AUTENTICADO`, `CONTA_EXCLUIDA_LGPD`. Senhas e corpo das requisições **nunca** são gravados.

**3) LGPD** – Páginas de termos e política; o cadastro exige `acceptTerms: true` e salva a data do consentimento; direitos do titular: `GET /api/users/me` (acesso) e `DELETE /api/users/me` (eliminação).

**4) API externa** – consulta de CEP via ViaCEP. Veja o projeto lógico em `docs/INTEGRACAO_API_EXTERNA.md`.

## Testando (curl ou Postman)
```bash
# 1. Cadastro (com consentimento LGPD)
curl -X POST http://localhost:8080/api/auth/register -H "Content-Type: application/json" \
  -d '{"name":"Maria","email":"maria@teste.com","password":"Senha1234","acceptTerms":true}'

# 2. Login (copie o "token" da resposta)
curl -X POST http://localhost:8080/api/auth/login -H "Content-Type: application/json" \
  -d '{"email":"maria@teste.com","password":"Senha1234"}'

# 3. Rota protegida
curl http://localhost:8080/api/users/me -H "Authorization: Bearer SEU_TOKEN"

# 4. API externa
curl http://localhost:8080/api/external/cep/01001000 -H "Authorization: Bearer SEU_TOKEN"

# 5. Autorização: USER recebe 403
curl -i http://localhost:8080/api/audit -H "Authorization: Bearer SEU_TOKEN"

# 6. Login como ADMIN e consulta dos logs
curl -X POST http://localhost:8080/api/auth/login -H "Content-Type: application/json" \
  -d '{"email":"admin@projeto.com","password":"Admin@12345"}'
curl http://localhost:8080/api/audit -H "Authorization: Bearer TOKEN_DO_ADMIN"
```
No Windows/PowerShell prefira o **Postman** (ou `curl.exe`).
Você também pode ver as tabelas em http://localhost:8080/h2-console (JDBC URL `jdbc:h2:file:./data/segurancadb`, usuário `sa`, senha vazia).

## O que mostrar ao orientador (validação)
1. Cadastro sem `acceptTerms` retornando erro 400.
2. Login válido e inválido (401).
3. Rota de ADMIN com usuário comum (403) e com ADMIN (200).
4. Tabela `audit_logs` com os eventos acima.
5. Coluna `password_hash` no H2 mostrando o hash BCrypt.
6. Páginas de termos e política.
7. Documento `docs/INTEGRACAO_API_EXTERNA.md` + consulta de CEP funcionando.

## Integrando ao seu projeto existente
Se você já tem um projeto Spring Boot: copie os pacotes `security`, `audit`, `config`, `model`, `repository`, `service`, `controller`, `dto`, `exception` e `util` (ajuste o `package`), copie `static/*.html` e as propriedades `app.*` do `application.properties`, e adicione as dependências do `pom.xml` (security, jpa, validation, jjwt).

## Segurança – antes de publicar
- Defina `JWT_SECRET` (Base64, ≥ 32 bytes) como variável de ambiente (**Run > Edit Configurations > Environment variables**).
- Troque a senha do ADMIN e desative `spring.h2.console.enabled`.
- Não versione a pasta `data/`.
- Revise os textos de LGPD (marcados com **[ADAPTE]**) com seu orientador.
