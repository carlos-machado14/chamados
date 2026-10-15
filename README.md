# Chamados

Plataforma SaaS multiempresa para abertura e acompanhamento de chamados, com conexão independente de cada empresa ao GitHub Issues e GitHub Projects, notificações configuráveis e domínios personalizados.

## Visão do produto

- **Operadora do SaaS:** gerencia tenants, instalações, auditoria e saúde da plataforma.
- **Empresa (tenant):** cria sua equipe, configura GitHub App, repositório e Project, identidade e DNS.
- **Cliente:** entra pelo domínio da empresa, abre chamados, envia anexos, acompanha status e recebe apenas atualizações autorizadas.
- **GitHub:** registra Issues e vincula itens ao Projects; nunca é acessado diretamente pelo cliente.

## Arquitetura inicial

- **Frontend:** Next.js + TypeScript + Tailwind CSS.
- **Backend:** NestJS + TypeORM + PostgreSQL.
- **Arquivos:** MinIO.
- **Fila:** Redis/BullMQ para criação de Issues, webhooks, retries, emails e sincronização.
- **Autenticação:** sessão com cookies HttpOnly/Secure, autorização por tenant e papel.
- **Deploy:** Docker + Dokploy, proxy reverso com HTTPS e roteamento por Host.
- **GitHub:** GitHub App com instalação e permissões mínimas; REST para Issues, GraphQL para Projects v2.

## Fluxo

1. Empresa é criada e recebe subdomínio da plataforma.
2. Administrador conecta o GitHub via instalação do GitHub App, escolhe repositório de Issues e Project.
3. Cliente abre chamado no portal; chamado é salvo com `tenant_id` e criado um job transacional via outbox.
4. Worker cria Issue e adiciona o item ao Project; armazena IDs externos.
5. Eventos do GitHub são autenticados, deduplicados e correlacionados; sincronização periódica cobre lacunas.
6. Alterações são convertidas em estados públicos, auditadas e notificadas segundo regras do tenant.
7. Comentários internos do GitHub **não** são publicados ao cliente por padrão; mensagens públicas são escritas pelo portal com classificação explícita.

## Regras fundamentais

- Todas as leituras e escritas de dados de negócio verificam tenant e papel no backend.
- Toda associação de domínio, usuário, instalação, repo, Project e ticket pertence a um único tenant.
- Nunca confiar no tenant_id enviado no corpo pelo cliente; resolver via sessão e host validado.
- Não usar uma credencial GitHub global de usuário para todos os tenants.
- Status técnico do Projects é mapeado para status público configurável.
- Suportar conta pessoal e organização, sem assumir eventos Projects v2 em contas pessoais.
- Criar chaves únicas para `installation_id`, `repository_id`/`issue_number` e `project_item_id` onde aplicável.
- Respeitar LGPD, controlar acesso a anexos, auditar mudanças e impedir vazamento cross-tenant.

## Documentação

- [Arquitetura e entregas](docs/ARQUITETURA.md)

## Status

**Planejamento técnico inicial.** Ainda não há frontend, backend, banco, deploy, GitHub App ou DNS implementados.
