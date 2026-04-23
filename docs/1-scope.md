# Fase 1 — PM/Scope: Securo

**Data:** 2026-04-23
**Agente:** PM/Scope (Claude Sonnet 4.6)
**Status do produto:** Production (sem monetização)
**Health:** https://financas.nanuck.com.br → frontend OK, `/api/health` retorna `{"status":"healthy"}`

---

## O que é

Finance manager self-hosted, forkado de projeto open-source (AGPL-3.0). Gerencia múltiplas contas bancárias, categoriza transações automaticamente por regras, controla budgets, goals e patrimônio líquido. Suporta multi-moeda com FX automático. 5 usuários reais em produção.

**Domínio:** https://financas.nanuck.com.br
**Repo:** /root/repos/securo
**Licença:** AGPL-3.0 (requer cuidado ao comercializar — usuários SaaS devem receber código-fonte)

---

## Público-alvo

- Profissionais brasileiros com múltiplas contas/investimentos que querem controle financeiro além do bancário
- Famílias com necessidade de budget compartilhado e tracking de metas (casa própria, aposentadoria)
- Micro-empreendedores individuais que misturam contas pessoal e PJ
- Perfil: 28–50 anos, renda média-alta, já usaram/abandonaram Excel ou Mobills/Organizze por falta de features avançadas

---

## Stack implementada

**Backend:** FastAPI + SQLAlchemy + Alembic + Celery + FastAPI-Users + 2FA TOTP
**Frontend:** React + TypeScript + Vite + Tailwind CSS + i18n (locales)
**Database:** PostgreSQL 16 + Redis 7 (queue Celery)
**Auth:** FastAPI-Users com TOTP 2FA implementado
**Integração:** Pluggy (bank sync — OFX, QIF, CAMT, CSV), Open Exchange Rates (FX)

---

## Rotas de API (backend/app/main.py)

| Módulo | Rota | Status |
|--------|------|--------|
| Auth custom | `/api/auth` | Login/logout + 2FA |
| Two-factor | `/api/auth/2fa` | TOTP QR + verify |
| Users | `/api/users` | Perfil, update |
| Accounts | configurado | Contas bancárias |
| Transactions | configurado | Lançamentos |
| Import | configurado | OFX/QIF/CAMT/CSV |
| Import logs | configurado | Histórico de importações |
| Recurring | configurado | Transações recorrentes |
| Budgets | configurado | Orçamentos por categoria |
| Goals | configurado | Metas financeiras |
| Assets | configurado | Patrimônio (imóveis, investimentos) |
| Asset values | configurado | Valor histórico de ativos |
| Categories | configurado | Categorias de transação |
| Category groups | configurado | Grupos de categoria |
| Rules | configurado | Categorização automática por regra |
| Dashboard | configurado | KPIs consolidados |
| Reports | configurado | Relatórios |
| Search | configurado | Busca de transações |
| Currencies | configurado | Moedas disponíveis |
| FX rates | configurado | Taxas de câmbio |
| Attachments | configurado | Comprovantes de transação |
| Payees | configurado | Beneficiários/pagadores |
| Connections | configurado | Bank connections (Pluggy) |
| Export | configurado | Exportação de dados |
| Settings | configurado | Configurações do app |
| Admin | configurado | Gestão de usuários, registro on/off |
| Health | `/api/health` | UP |

---

## Modelos de dados (19 tabelas)

| Modelo | Descrição |
|--------|-----------|
| User | Auth + perfil (FastAPI-Users UUID) |
| Account | Contas bancárias, saldo, moeda |
| Transaction | Lançamentos com categoria, conta, payee |
| Category | Categorias de despesa/receita |
| CategoryGroup | Agrupamento de categorias |
| Rule | Regras de categorização automática |
| Budget | Orçamento por categoria e período |
| Goal | Metas financeiras com progresso |
| Asset | Patrimônio (imóveis, aplicações) |
| AssetValue | Histórico de valores de ativos |
| FxRate | Taxas de câmbio por data |
| BankConnection | Integração Pluggy por usuário |
| RecurringTransaction | Transações periódicas |
| TransactionAttachment | Arquivos vinculados a transações |
| ImportLog | Histórico de importações |
| Payee | Beneficiários e fornecedores |
| PayeeMapping | Mapeamento de payees para categorias |
| AppSetting | Configurações globais do sistema |
| alembic_version | Controle de migração |

---

## Páginas de frontend (frontend/src/pages/)

| Rota | Status |
|------|--------|
| `/login` | Implementado |
| `/register` | Implementado |
| `/setup` | Configuração inicial (first-run) |
| `/dashboard` | KPIs, gráficos, saldo consolidado |
| `/accounts` | Listar e gerir contas |
| `/account-detail` | Detalhe de conta com transações |
| `/transactions` | Extrato completo com filtros |
| `/recurring` | Transações recorrentes |
| `/import` | Upload OFX/QIF/CAMT/CSV |
| `/categories` | Gestão de categorias |
| `/budgets` | Orçamentos por período |
| `/goals` | Metas financeiras |
| `/assets` | Patrimônio líquido |
| `/reports` | Relatórios de receita/despesa |
| `/rules` | Regras de categorização |
| `/payees` | Beneficiários |
| `/admin` | Painel administrativo |

---

## Fluxos críticos

### 1. Onboarding de novo usuário
```
/register → email verify → /setup (moeda padrão, conta inicial)
→ /dashboard → importar extrato (OFX/CSV) → categorizar automaticamente
→ configurar budgets e goals
```

### 2. Fluxo de importação bancária
```
/import → upload OFX/QIF/CAMT/CSV → parse backend → preview transações
→ aplicar regras automáticas → confirmar → lançar em /transactions
```

### 3. Dashboard de patrimônio
```
/dashboard → saldo consolidado por conta → evolução de NetWorth
→ receitas vs despesas → budgets consumidos vs planejado
→ progresso de goals
```

### 4. Sync bancário (Pluggy)
```
/accounts → conectar banco (OAuth Pluggy) → Celery task sync_all_connections
→ importar transações automaticamente → aplicar regras → atualizar saldo
```

---

## O que está implementado

- Backend com 25+ rotas funcionais cobrindo todo o ciclo financeiro pessoal
- 2FA TOTP com QR code
- Rate limiting: login, register, password reset
- Regras de categorização automática por payee/descrição
- Transações recorrentes com Celery
- Suporte a multi-moeda com FX automático
- Import de extratos: OFX, QIF, CAMT.053, CSV
- Exportação de dados
- Patrimônio líquido com histórico de valores de ativos
- Frontend completo com 16 páginas
- Admin panel para gestão de usuários
- Health endpoint `/api/health`
- Celery + Redis para tasks assíncronas

---

## O que falta

- **Monetização zero:** sem Stripe, sem planos, sem billing — usuários usam gratuitamente
- **Branding próprio:** app ainda carrega identidade do projeto original (nome, logo, paleta); sem identidade "Securo"
- **LGPD incompleto:** direito ao esquecimento não implementado; sem endpoint de exportação de dados pessoais para compliance
- **Notificações ausentes:** sem alertas de budget excedido, transação suspeita, vencimento de recorrente
- **Backups automatizados:** sem estratégia de backup geográfico documentada ou implementada
- **Pluggy não ativado em produção:** integração existe no código mas sem credenciais configuradas
- **Open Exchange Rates não ativo:** FX manual apenas
- **Multi-tenancy limitado:** sem workspace compartilhado (casal, família, sócio)
- **Mobile experience:** não é PWA, responsividade básica apenas

---

## 3 maiores gaps para launch (SaaS comercial)

### GAP 1 — Ausência total de monetização (BLOQUEADOR CRÍTICO)
Não existe nenhuma camada de billing: sem planos, sem Stripe, sem gate de features por tier. Os 5 usuários atuais não pagam. Para lançar como SaaS: implementar Stripe Billing com planos recorrentes (sugestão: Básico R$9,90/mês — 1 conta, Personal R$29,90/mês — ilimitado, Família R$49,90/mês — multi-usuário). Exige: modelo de `Subscription` no DB, middleware de verificação de tier, webhook Stripe, página de pricing.

### GAP 2 — Branding e identidade "Securo" inexistente
O produto ainda usa nome, paleta e componentes do projeto open-source original. Usuários novos não reconhecem o produto como "Securo". Ação: rebrand completo — nome em todas as telas, logo, paleta acra, favicon, título de página, emails transacionais, domínio principal, OG tags para compartilhamento.

### GAP 3 — LGPD incompleto bloqueia operação legal como SaaS brasileiro
Sem implementação de direito ao esquecimento (`DELETE /api/users/me/data`) e sem endpoint de exportação de dados pessoais (`GET /api/users/me/export`), operar como SaaS com cobrança expõe a multas da ANPD. Ação: implementar os dois endpoints, adicionar política de privacidade na UI, registrar DPO, criar log de acesso a dados pessoais.

---

## Decisões de escopo

1. Manter base AGPL — distribuir código-fonte ao usar SaaS (obrigação de licença) ou migrar para licença dual (open-source core + SaaS proprietary)
2. Pluggy como única integração bancária inicial (OFX/CSV como fallback)
3. Plano Família como diferencial competitivo (Organizze/Mobills não têm workspace multi-usuário real)
4. Foco em mercado BR: BRL primário, suporte a USD/EUR secundário

---

## Próxima fase

**Fase 2 — Competitive Intel:** mapear Organizze, Mobills, GuiaBolso, Minhas Economias, Toshl — pricing, features, diferenciais do Securo.
