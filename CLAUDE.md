# securo

Este repositório é um produto da Fábrica de SaaS Nanuck.

**Pipeline e regras:** https://github.com/nanuckagent/saas-constructor
**Arquitetura, ADRs, skills:** `/root/saas-constructor/`
**Vault de conhecimento:** `/root/saas-constructor/knowledge/products/securo.md`

## Regras não-negociáveis herdadas da plataforma

1. Triple sync ao final de cada fase: GitHub + Obsidian (via pull) + Notion (MCP)
2. `ip-whitelist-admin` NUNCA em rotas `/api` ou `/v1` (apenas `/metrics`)
3. HEALTHCHECK sempre `127.0.0.1`, nunca `localhost`
4. Design ACRA: sidebar #1a2d7a, primary #3b5bdb, Inter, cards rounded-2xl
5. Auth passwordless (ADR-005), Stripe para pagamentos (ADR-002)
6. Build real + `curl` 200 antes de declarar PASS

## Estrutura de docs (14 fases do pipeline)

Ver `docs/README.md` para o mapa completo.
