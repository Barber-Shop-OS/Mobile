# Backlog — Mobile BarberOS (App)

> Documento vivo, escrito para ser executado por um agente de IA. Cada tarefa é autocontida e tem critério de aceite verificável.
> Última revisão: 2026-11-07. Estado: **repositório vazio** (só `README.md`, commit `d9e7b6b`).
> Backlogs irmãos: Back-End `../../Back-End/docs/BACKLOG.md` (`BE-xx`) · Segurança `../../Back-End/docs/SECURITY_BACKLOG.md` (`SEC-xx`) · Front `../../Front-End/docs/BACKLOG.md` (`FE-xx`).

---

## 0. Como usar

### Regras
- Ordem: P0 → P1 → P2 → P3; respeitar **Depende de**. Se depender de `BE-xx` não feito, fazer o BE primeiro — não mockar para contornar (mocks só em teste).
- **Um commit por tarefa:** `feat(MOB-07): wizard de agendamento`.
- **[DECISÃO PENDENTE]** = pedir "ok" ao responsável; a recomendação está escrita.

### Decisões e premissas
| Data | Decisão |
|---|---|
| 2026-10-23 | Sem app iOS no curto prazo; usuários de iPhone usam o web (FE). **O app mira Android primeiro.** A stack escolhida permite publicar iOS depois sem reescrever. |
| 2026-11-07 | Planos R$ 99/199/399 via AbacatePay. **O app não vende assinatura** (ver MOB-16). |
| 2026-11-07 | Regras de negócio iguais ao web: agendamento nasce `confirmado`; cliente cancela/reagenda até 24 h antes (`can_cancel_until`, `allowed_actions[]` vêm da API). |

### Escopo do app (por perfil)
- **Cliente (MVP):** buscar barbearia, ver perfil/filial, agendar, meus agendamentos, cancelar/reagendar, lembretes push, perfil.
- **Barbeiro (MVP):** agenda do dia/semana, próximo cliente, concluir / não compareceu / registrar pagamento, bloqueios próprios, métricas do período, push de novo agendamento/cancelamento.
- **Dono (P2, versão leve):** agenda da filial, novo agendamento de balcão, dashboard resumido. **Configuração pesada (filiais, serviços, planos, convites) continua no web** — o app abre o web nesses pontos.

### Stack recomendada [DECISÃO PENDENTE — recomendação já assumida nas tarefas]
- **React Native + Expo (SDK estável mais recente) + TypeScript**, `expo-router` (rotas por arquivo), **Redux Toolkit + RTK Query** (mesmo padrão do web — FE-05), Axios só se necessário (RTK Query `fetchBaseQuery` basta), `react-hook-form` + `zod` (mesmos schemas do web), `dayjs` com utc/timezone, `expo-secure-store`, `expo-notifications`, `expo-linking`, EAS Build/Submit/Update, Jest + `@testing-library/react-native` + `msw`, Maestro para E2E.
- Motivo: a equipe já trabalha em React/TS/Redux no web; tipos gerados do mesmo `openapi.json` (BE-43); um código para Android e iOS.

### Convenções
1. Estrutura: `app/` (rotas expo-router por grupo: `(public)`, `(auth)`, `(customer)`, `(barber)`, `(owner)`), `src/features/<dominio>/{api,components,hooks,types.ts}`, `src/components/ui/*`, `src/services/baseApi.ts`, `src/utils/{date,money}.ts`.
2. Tipos da API gerados (`npm run gen:api` a partir de `Back-End/docs/openapi.json`). Nunca `any`.
3. Toda tela com dados: loading (skeleton), vazio, erro com "tentar novamente", offline.
4. Datas: API em UTC, exibição `pt-BR` no fuso da filial.
5. Acessibilidade: `accessibilityLabel` em todo botão de ícone, alvos ≥ 44 pt, suporte a fonte grande.
6. Nenhum segredo no bundle; env por `app.config.ts` + `EXPO_PUBLIC_API_URL` apenas.

### Definição de pronto (DoD)
- [ ] Critérios de aceite atendidos em emulador Android (API 26+) e um aparelho físico.
- [ ] `npm run typecheck`, `npm run lint`, `npm test` verdes.
- [ ] Estados de loading/vazio/erro/offline implementados.
- [ ] Sem dados fixos (mock) fora de testes.
- [ ] `[x]` marcado aqui.

---

## Fase P0 — Fundação

### [ ] MOB-01 — Organizar o repositório e criar o projeto
- **Depende de:** decisão de stack
- **Fazer:** `npx create-expo-app` (template TypeScript + expo-router) na raiz; `README.md` com setup (Node LTS, Android Studio/emulador, Expo Go, variáveis); `.gitignore` (node_modules, `.expo`, builds, `.env*` exceto `.env.example`); `.env.example` com `EXPO_PUBLIC_API_URL`; `app.config.ts` com `name`, `slug`, `scheme: "barberos"`, `android.package` (ex.: `com.barberos.app`) [DECISÃO PENDENTE: nome/pacote definitivos].
- **Aceite:** `npx expo start` abre a tela inicial no emulador Android.

### [ ] MOB-02 — Tooling, testes e CI
- **Depende de:** MOB-01
- **Fazer:** ESLint (config do Expo + `react-hooks` + `jsx-a11y` equivalente RN) + Prettier; Jest (`jest-expo`) + Testing Library + `msw`; scripts `typecheck`, `lint`, `test`; GitHub Actions: `npm ci && typecheck && lint && test`; `npm audit` e `gitleaks` no CI `[SEC-16, SEC-14]`; Dependabot.
- **Aceite:** CI verde em PR.

### [ ] MOB-03 — Camada de dados, tipos e utilitários
- **Depende de:** MOB-02, BE-43 (até lá, tipos escritos à mão em `src/types/api.ts` seguindo o OpenAPI atual)
- **Fazer:** `src/services/baseApi.ts` (RTK Query, `baseUrl = EXPO_PUBLIC_API_URL + '/api'`, injeta `Authorization`, trata 401 com refresh — MOB-05), `tagTypes` iguais aos do web; `src/utils/date.ts`, `src/utils/money.ts` (`Intl` pt-BR/BRL — usar polyfill `intl` se o Hermes não cobrir timezones); tradução de `code` de erro do backend (BE-37) para mensagens.
- **Aceite:** chamada `GET /api/plans` tipada funcionando em tela de debug (remover depois).

### [ ] MOB-04 — Kit de UI e design tokens
- **Depende de:** MOB-01
- **Fazer:** tokens (cores alinhadas ao web — primária `#0b4bd8` como `primary`), tipografia, espaçamento; componentes `Button`, `TextField`, `Select`/bottom sheet, `Card`, `Badge` de status de agendamento, `Skeleton`, `EmptyState`, `ErrorState`, `Toast`, `ConfirmSheet`, `Avatar`; tema claro (escuro em P3).
- **Aceite:** tela de catálogo dos componentes no modo dev.

### [ ] MOB-05 — Autenticação e sessão segura `[SEC-19]`
- **Depende de:** MOB-03, BE-02 (refresh: BE-30)
- **Fazer:** telas login, cadastro de cliente (com aceite de termos — BE-40), "esqueci a senha" (BE-26, abre fluxo por e-mail com deep link para `barberos://reset?token=`); armazenar tokens **só** em `expo-secure-store`; bootstrap de sessão (`GET /api/me`); refresh automático com fila única de requisições (BE-30); logout limpa store + secure store + cache RTK Query; roteamento por papel (`cliente` → `(customer)`, `barbeiro` → `(barber)`, `owner` → `(owner)`; usuário com múltiplos papéis escolhe o modo).
- **Aceite:** matar o app e reabrir mantém a sessão; 401 após refresh falho volta ao login; nada sensível em `AsyncStorage`.

### [ ] MOB-06 — Navegação, deep links e links universais
- **Depende de:** MOB-05
- **Fazer:** grupos de rota protegidos por papel; deep links `barberos://b/:slug`, `barberos://convite/:token`, `barberos://agendamentos/:id`; Android App Links para `https://<dominio>/b/:slug` (arquivo `assetlinks.json` servido pelo web — tarefa no FE) [DECISÃO PENDENTE: domínio]; preservar destino ao exigir login (agendar → login → volta à revisão).
- **Aceite:** abrir link da barbearia no WhatsApp com o app instalado cai no perfil dela.

---

## Fase P1 — Cliente

### [ ] MOB-07 — Busca de barbearias
- **Depende de:** MOB-04, BE-25
- **Fazer:** busca por texto/cidade/serviço, "perto de mim" (`expo-location`, permissão pedida só ao tocar), lista paginada com scroll infinito, cards com nome, filial, distância, "aberto agora".
- **Aceite:** permissão negada mantém a busca por cidade funcionando.

### [ ] MOB-08 — Perfil da barbearia e da filial
- **Depende de:** MOB-07, BE-18, BE-10
- **Fazer:** `GET /api/barbershop/slug/:slug`: capa/logo, descrição, filiais, serviços (preço/duração), barbeiros; filial com horários (dia atual destacado), botões "Como chegar" (intent de mapas), "Ligar", "WhatsApp"; CTA "Agendar" pré-preenche o wizard.
- **Aceite:** slug inexistente → tela "barbearia não encontrada".

### [ ] MOB-09 — Wizard de agendamento
- **Depende de:** MOB-08, BE-13, BE-14, BE-15
- **Fazer:** filial → serviço → profissional ("sem preferência") → data (30 dias, fechados desabilitados) → horário (`/availability`) → revisão com política "cancelamento gratuito até 24 h antes" e observação → confirmar com header `Idempotency-Key` (UUID por tentativa) → sucesso com "adicionar ao calendário" (`expo-calendar`, permissão opcional). `409 SLOT_TAKEN` → aviso e volta aos horários atualizados; `409 TOO_MANY_ACTIVE_APPOINTMENTS` → mensagem clara.
- **Testes:** componente com `msw` cobrindo sucesso, 409 e erro de rede (retry não duplica graças à idempotency key).
- **Aceite:** fluxo completo em ≤ 6 toques após escolher a barbearia.

### [ ] MOB-10 — Meus agendamentos e detalhes
- **Depende de:** MOB-09, BE-20, BE-15
- **Fazer:** abas Próximos/Histórico (`/me/appointments`), detalhe com status, endereço, ações conforme `allowed_actions` (cancelar com motivo, reagendar reaproveitando os passos de data/horário), texto "Você pode cancelar até dd/mm hh:mm"; após o prazo, só "Falar com a barbearia". Pull-to-refresh.
- **Aceite:** ação ausente em `allowed_actions` nunca aparece.

### [ ] MOB-11 — Push notifications
- **Depende de:** MOB-05, BE-45
- **Fazer:** `expo-notifications`: pedir permissão depois do primeiro agendamento (não no primeiro acesso); registrar token em `POST /api/me/push-tokens`, remover no logout; canal Android "Lembretes"; tocar na notificação abre o agendamento (deep link).
- **Aceite:** lembrete de 2 h chega e abre o detalhe correto.

### [ ] MOB-12 — Perfil do cliente
- **Depende de:** MOB-05, BE-29, BE-40
- **Fazer:** editar nome/telefone, trocar senha, trocar e-mail (com reverificação), preferências de notificação, termos/privacidade, exportar dados, excluir conta (confirmação dupla), sair.
- **Aceite:** exclusão desloga e impede novo login.

---

## Fase P1 — Barbeiro

### [ ] MOB-13 — Agenda do barbeiro
- **Depende de:** MOB-05, BE-15, BE-11
- **Fazer:** visão dia (lista/timeline) e semana; próximo cliente em destaque (nome, serviço, horário, observação, telefone); ações rápidas: concluir, não compareceu, cancelar (motivo), registrar pagamento (PIX/dinheiro/cartão — BE-07); atualização ao focar o app + polling 60 s; push "novo agendamento"/"cancelado".
- **Aceite:** usável com uma mão; ações respeitam `allowed_actions`.

### [ ] MOB-14 — Bloqueios e métricas do barbeiro
- **Depende de:** MOB-13, BE-11, BE-24
- **Fazer:** criar/remover bloqueio (almoço, folga) com aviso de conflito (409 com lista); métricas do período (atendimentos, receita, faltas) via `/api/dashboard/barber`.
- **Aceite:** bloqueio sobre agendamento existente mostra os conflitos e não salva.

### [ ] MOB-15 — Aceitar convite de barbeiro
- **Depende de:** MOB-06, BE-22
- **Fazer:** deep link `barberos://convite/:token` → mostra barbearia/filial → login com o mesmo e-mail ou criação de conta → aceite → modo barbeiro. Token expirado/usado → mensagem e contato do dono.
- **Aceite:** convite aceito leva direto à agenda.

---

## Fase P2 — Dono (versão leve) e loja

### [ ] MOB-16 — Modo dono: agenda, balcão e resumo
- **Depende de:** MOB-13, BE-24, BE-23
- **Fazer:** seletor barbearia/filial; agenda da filial (colunas por barbeiro na semana em tablet, lista no celular); novo agendamento de balcão (busca de cliente + cliente rápido + slots); cards do dashboard (receita, agendamentos, ocupação). Banner de status da assinatura vindo de `/api/me` (`billing_banner`).
- **Assinatura e configuração:** botões "Gerenciar plano", "Filiais", "Serviços", "Equipe" abrem o **web** no navegador do sistema. Não exibir preços nem fluxo de compra dentro do app (evita conflito com as regras de cobrança da Google Play/App Store para bens digitais; o plano é um serviço B2B contratado no site) [DECISÃO PENDENTE — revisar a política vigente da loja antes da publicação].
- **Aceite:** owner sem assinatura ativa vê o aviso e o link para o web; nenhuma tela de compra no app.

### [ ] MOB-17 — Builds, ambientes e publicação Android
- **Depende de:** MOB-12, MOB-15
- **Fazer:** EAS Build com perfis `development`, `preview` (APK interno apontando para staging) e `production` (AAB); EAS Update (OTA) com canal por ambiente; versionamento automático (`runtimeVersion` por versão nativa); conta Google Play Console, ficha da loja (descrição pt-BR, screenshots, ícone, feature graphic), **política de privacidade pública** (URL do web), formulário de segurança de dados (Data safety) coerente com SEC-20, classificação de conteúdo, teste fechado com 12+ testadores por 14 dias (exigência atual para contas pessoais novas — confirmar) antes da produção.
- **Aceite:** build de produção publicado no teste fechado.

### [ ] MOB-18 — Atualização obrigatória e configuração remota `[SEC-19]`
- **Depende de:** MOB-17, BE (endpoint `GET /api/app-config` — criar junto, ver BE-48)
- **Fazer:** ao abrir, consultar versão mínima suportada; abaixo dela → tela bloqueante "Atualize o app" com link da loja; flags simples (ex.: desligar agendamento em incidente).
- **Aceite:** subir `min_version` no backend bloqueia versões antigas.

---

## Fase P3 — Qualidade e expansão

### [ ] MOB-19 — Testes E2E
- **Depende de:** MOB-10, MOB-13, BE-39
- **Fazer:** Maestro: (1) cadastro cliente → agendar → ver em "Meus agendamentos" → cancelar; (2) barbeiro login → concluir atendimento → registrar pagamento; (3) convite de barbeiro. Rodar contra backend com seed em CI (emulador) ou manual por release.
- **Aceite:** 3 fluxos verdes antes de cada release de produção.

### [ ] MOB-20 — Offline, desempenho e robustez
- **Depende de:** MOB-13
- **Fazer:** banner offline (`@react-native-community/netinfo`); cache da última agenda do barbeiro para leitura offline; ações em fila **não** (agendamento exige servidor — mostrar erro); imagens com cache (`expo-image`); medir tempo de abertura (< 2 s em aparelho médio); Sentry (`sentry-expo`/`@sentry/react-native`) com scrubbing `[SEC-29]`.
- **Aceite:** sem internet, agenda do dia já carregada continua visível.

### [ ] MOB-21 — Acessibilidade e polimento
- **Depende de:** telas P1
- **Fazer:** TalkBack em todos os fluxos P1; fonte grande sem cortar layout; contraste AA; modo escuro; haptics nas ações principais; ícone adaptativo e splash.
- **Aceite:** fluxo de agendamento completo com TalkBack.

### [ ] MOB-22 — Publicação iOS [FUTURO / DECISÃO PENDENTE]
- **Depende de:** decisão de produto (hoje: iPhone usa o web)
- **Fazer quando aprovado:** conta Apple Developer, EAS Submit para TestFlight, Universal Links (`apple-app-site-association` no web), revisão de política de compras (mesma regra de MOB-16), testes em iPhone com notch/safe areas.

---

## Sequência recomendada
1. **Sprint M1 (fundação):** MOB-01 → MOB-06 — pode começar em paralelo com a sprint A do backend (só precisa de BE-02 para login).
2. **Sprint M2 (cliente):** MOB-07 → MOB-12 — depende das sprints B/C do backend (disponibilidade e agendamento).
3. **Sprint M3 (barbeiro):** MOB-13 → MOB-15.
4. **Sprint M4 (loja):** MOB-17, MOB-18, MOB-19 → teste fechado na Play Store.
5. **Depois:** MOB-16, MOB-20 → MOB-22.

> Recomendação de priorização do negócio: o **web é o canal de lançamento** (dono assina e configura no web; cliente agenda pelo link). O app entra depois que o fluxo web estiver faturando, começando pelo barbeiro (uso diário) e cliente (lembretes push reduzem faltas).
